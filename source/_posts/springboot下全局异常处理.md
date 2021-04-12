---
title: springboot下全局异常处理
date: 2020-06-20 16:37:10
categories: [springboot]
tags: [异常,spring]
---

想了一堆的废话，写了又删，就是一个破技术了，没那么多的废话，只写点干货得了。

统一的异常处理，能少些点冗余得代码，本质上还是AOP在起作用。首先自定义异常继承于**RuntimeException**,代码中使用了**lombok** 

``` java
// 自定义异常类型
@Getter
public class LyException extends RuntimeException{
    private ExceptionEnum exceptionEnum;
    public LyException(ExceptionEnum exceptionEnum){
        this.exceptionEnum=exceptionEnum;
    }
}
```

统一异常处理

``` java
@Slf4j
@ControllerAdvice // 注解的作用还挺多的，异常处理仅仅是一个，有时间再记一下
public class BasicExceptionHandler {
    @ExceptionHandler(LyException.class) // 绑定这个异常
    public ResponseEntity<ExceptionResult> handleException(LyException e){
        return ResponseEntity.status(e.getExceptionEnum().value()).body(new ExceptionResult(e.getExceptionEnum()));  // 返回给前端JSON格式数据，方便的写法
    }
}
```

**@ExceptionHandler** 异常捕获的具有就近原则，程序中报**NumberFormatException**的时候，注解中分别修饰**NumberFormatException** 、**Exception** 当然优先进入**NumberFormatException**的方法中。

异常处理方法体中封装的对象，更方便错误信息表示：

``` java
@Data
public class ExceptionResult {
    private int status;
    private String message;
    private long timestamp;
    public ExceptionResult(ExceptionEnum em){
        this.status=em.value();
        this.message=em.message();
        this.timestamp=System.currentTimeMillis();
    }
}
```

异常信息的枚举类

``` JAVA
@NoArgsConstructor  // LOMBOK的构造参数注解
@AllArgsConstructor
public enum ExceptionEnum {
    DELETE_SPEC_GROUP_FAILED(500, "商品规格组删除失败"),
    UPDATE_SPEC_GROUP_FAILED(500, "商品规格组更新失败"),
    ;
    int value;
    String message;

    public int value(){
        return this.value;
    }
    public String message(){
        return this.message;
    }
}

```

使用中，依赖添加好以后，直接可以再service中抛出,而不用手工捕获的异常

``` java
@Service
public class CategoryServiceImpl implements CategoryService {
    
    @Autowired
    private CategoryMapper categoryMapper;
    
    @Override
    public List<Category> queryCategoryByPid(Long pid) {
        Category category = new Category();
        category.setParentId(pid);
        List<Category> categoryList = categoryMapper.select(category);
        if (CollectionUtils.isEmpty(categoryList)) {
            throw new LyException(ExceptionEnum.CATEGORY_NOT_FOUND);
        }
        return categoryList;
    }
    
    @Override
    public List<Category> queryCategoryByIds(List<Long> ids) {
        return categoryMapper.selectByIdList(ids);
    }

    @Override
    public List<Category> queryAllByCid3(Long id) {
        Category c3 = categoryMapper.selectByPrimaryKey(id);
        Category c2 = categoryMapper.selectByPrimaryKey(c3.getParentId());
        Category c1 = categoryMapper.selectByPrimaryKey(c2.getParentId());
        List<Category> list = Arrays.asList(c1, c2, c3);
        if (CollectionUtils.isEmpty(list)) {
            throw new LyException(ExceptionEnum.CATEGORY_NOT_FOUND);
        }
        return list;
    }
}
```

补充：异常处理也可以这种形式

``` java
// 放入ModelAndView中
@ControllerAdvice
    class GlobalExceptionHandler {
        public static final String DEFAULT_ERROR_VIEW = "error";
        @ExceptionHandler(value = Exception.class)
        public ModelAndView defaultErrorHandler(HttpServletRequest req, Exception e) throws Exception {
            ModelAndView mav = new ModelAndView();
            mav.addObject("exception", e);
            mav.addObject("url", req.getRequestURL());
            mav.setViewName(DEFAULT_ERROR_VIEW);
            return mav;
        }
    }
```

亦可，其实本质上都是一样的，只为清楚明了一点

``` java
@ControllerAdvice
public class GlobalExceptionHandler {
@ExceptionHandler(value = MyException.class)
@ResponseBody
public ErrorInfo<String> jsonErrorHandler(HttpServletRequest req, MyException e) throws Exception {
	ErrorInfo<String> r = new ErrorInfo<>();
	r.setMessage(e.getMessage());
	r.setCode(ErrorInfo.ERROR);
	r.setData("Some Data");
	r.setUrl(req.getRequestURL().toString());
	return r;
	}
}
```

