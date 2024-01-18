# 简介

YAML 是 "YAML Ain't Markup Language"（YAML 不是一种标记语言）的递归缩写。在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种标记语言）。

非常适合用来做以数据为中心的配置文件

# 基本用法

- `key: value`
    
    > key和value之间有空格
    
- 大小写敏感
    
- 使用缩进表示层级关系
    
- 缩进不允许使用tab，只允许空格，但在idea上使用tab也可以
    
- 缩进的空格数不重要，只要相同层级的元素左对齐即可
    
- `#`表示注释
    
- 字符串无需加引号，如果要加，单引号会被转义，双引号不会不转义
    

# 数据类型

## 字面量

单个的、不可再分的值。date、boolean、string、number、null

yaml

k: v

## 对象

键值对的集合。map、hash、set、object

yaml

#行内写法：
k: {k1:v1,k2:v2,k3:v3}
#或
k: 
  k1: v1
  k2: v2
  k3: v3

## 数组

一组按次序排列的值。array、list、queue

yaml

#行内写法： 
k: [v1,v2,v3]
#或者
k:
 - v1
 - v2
 - v3

## 示例

java

@Data
public class Person {
    
    private String userName;
    private Boolean boss;
    private Date birth;
    private Integer age;
    private Pet pet;
    private String[] interests;
    private List<String> animal;
    private Map<String, Object> score;
    private Set<Double> salarys;
    private Map<String, List<Pet>> allPets;
}

@Data
public class Pet {
    private String name;
    private Double weight;
}

yml

# yaml表示以上对象
person:
  userName: zhangsan
  boss: false
  birth: 2019/12/12 20:12:33
  age: 18
  pet: 
    name: tomcat
    weight: 23.4
  interests: [篮球,游泳]
  animal: 
    - jerry
    - mario
  score:
    english: 
      first: 30
      second: 40
      third: 50
    math: [131,140,148]
    chinese: {first: 128,second: 136}
  salarys: [3999,4999.98,5999.99]
  allPets:
    sick:
      - {name: tom}
      - {name: jerry,weight: 47}
    health: [{name: mario,weight: 47}]	

# 配置提示

自定义的类和配置文件绑定一般没有提示。

yml

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
</dependency>

 <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.springframework.boot</groupId>
                            <artifactId>spring-boot-configuration-processor</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
 </build>
```

***
GPT回答
YAML（YAML Ain't Markup Language）是一种用于配置文件和数据交换的人类可读的数据序列化标准。在Kubernetes中，YAML文件用于定义资源对象，如Pods、Deployments、Services等。以下是YAML基本语法的一些关键点：

1. **基础结构**
   - **缩进**：YAML使用缩进来表示层级关系。通常使用空格而不是制表符进行缩进。
   - **键值对**：YAML文件主要由键值对组成，用冒号后跟一个空格来分隔键和值。
     ```
     key: value
     ```

2. **列表和字典**
   - **列表**：用短横线“-”开始的行表示一个列表项。
     ```
     items:
       - item1
       - item2
     ```
   - **字典**：一个键后跟一个值的映射，可以嵌套。
     ```
     user:
       name: John Doe
       age: 30
     ```

3. **多行字符串**
   - **管道符号（|）**：保留换行符。
     ```
     description: |
       This is a description
       over multiple lines.
     ```
   - **大于号（>）**：折叠换行符，以一个空格替代。
     ```
     description: >
       This is a description
       over multiple lines.
     ```

4. **注释**
   - 使用井号“#”来添加注释。
     ```
     # 这是一条注释
     key: value
     ```

5. **数据类型**
   - **字符串**：通常直接写出来，也可以用双引号或单引号包围。
   - **数字**：直接写出数字，区分整数和浮点数。
   - **布尔值**：使用`true`或`false`。

6. **特殊结构**
   - **锚点和别名**：允许你定义一个锚点（&），并在文件的其他地方以别名（*）重复使用。
     ```
     base: &base
       name: Everyone has same name

     foo:
       <<: *base
       age: 10
     ```

7. **文件合并**
   - 多个YAML文档可以在同一个文件中，每个文档由三个连字符“---”分隔。

YAML语法看似简单，但缩进和格式的准确性非常重要，因为这会直接影响配置的解析。建议在编写YAML文件时使用文本编辑器的YAML模式，这可以帮助检查语法错误。在用于Kubernetes时，理解如何通过YAML文件定义资源是非常重要的。

***
#问：k8s中用yaml如何定义Pods、Deployments、Services
在Kubernetes（k8s）中，Pods、Deployments和Services是非常核心的资源类型，它们通常通过YAML（YAML Ain't Markup Language）文件定义。我将为每种资源类型提供一个基本的YAML定义示例，并简要解释其关键部分。

### 1. 定义Pod

Pod是Kubernetes中最基本的部署单位，通常包含一个或多个容器。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: myapp
spec:
  containers:
  - name: my-container
    image: nginx
    ports:
    - containerPort: 80
```

- `apiVersion`: 定义使用的Kubernetes API版本，对于Pod通常是`v1`。
- `kind`: 资源类型，这里是`Pod`。
- `metadata`: 包含了Pod的元数据，如`name`和`labels`。
- `spec`: 定义Pod的具体规格。在这里，它定义了一个容器。
- `containers`: 容器的列表。每个容器需要一个名称（`name`）和镜像（`image`），这里使用了nginx镜像。
- `containerPort`: 容器暴露的端口号，这里是80。

### 2. 定义Deployment

Deployment用于管理多个Pod的副本，并确保指定数量的Pod副本同时运行。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: my-container
        image: nginx
        ports:
        - containerPort: 80
```

- `apiVersion`: 这里是`apps/v1`，因为Deployment属于apps组。
- `kind`: `Deployment`。
- `metadata`: 定义Deployment的名称和标签。
- `spec`: 
  - `replicas`: 声明需要的Pod副本数量。
  - `selector`: 用于选择哪些Pods将由此Deployment管理。
  - `template`: 定义Pod的模板。它的结构与单个Pod的定义类似。

### 3. 定义Service

Service定义了如何访问Pod，包括选择哪些Pod和如何访问它们（比如通过负载均衡）。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

- `apiVersion`: 对于Service，通常是`v1`。
- `kind`: `Service`。
- `metadata`: 定义Service的名称。
- `spec`: 
  - `selector`: 选择哪些Pod将被此Service路由。
  - `ports`: 定义端口配置。
    - `protocol`: 通信协议，通常是TCP。
    - `port`: Service的端口。
    - `targetPort`: Pod的端口。
  - `type`: Service的类型，这里是`LoadBalancer`，它会在集群外部提供一个可访问的IP。

这些YAML文件只是各种资源类型的基本示例。在实际应用中，可能需要更详细的配置，如环境变量、卷挂载、资源限制、就绪探针等。熟悉这些基本概念后，可以通过阅读Kubernetes官方文档来深入了解更多高级特性和配置选项。