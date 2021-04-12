---
title: maven的小tip
date: 2021-04-09 15:49:42
categories: [maven]
tags: [工具,tip]
---

# 1.全量打包（打入所有的依赖）

``` xml
<build> 
      <plugins> 
        <plugin> 
          <artifactId>maven-assembly-plugin</artifactId> 
          <configuration> 
            <archive> 
              <manifest> 
                <mainClass>com.allen.capturewebdata.Main</mainClass> 
              </manifest> 
            </archive> 
            <descriptorRefs> 
              <descriptorRef>jar-with-dependencies</descriptorRef> 
            </descriptorRefs> 
          </configuration> 
        </plugin> 
      </plugins> 
</build>
```

``` sh
mvn assembly:assembly #打包命令
```



