排除工具版本用<exclusions>

排除工具内引用jar包用

```
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi-ooxml</artifactId>
            <version>5.2.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.poi</groupId>
            <artifactId>poi</artifactId>
            <version>5.2.0</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```
