---



layout:		post



title:		初识 Mybatis（二）—— 动态 SQL



subtitle:	数据库模块



date:		2018-07-02



author:		L



header-img:	img/post/2018-07/02.jpg



tags:



    - Mybatis



---

### 正文之前

> 上一篇介绍了 [Mybatis 的基本用法](http://hxblog.site/2018/06/21/%E5%88%9D%E5%A7%8B-Mybatis-%E4%B8%80-%E5%9F%BA%E6%9C%AC%E9%85%8D%E7%BD%AE%E5%8F%8A%E7%94%A8%E6%B3%95/)，这一篇我们着重介绍一下 Mybatis 对 SQL 拼接的支持，在之前写 JDBC 代码的时候，SQL 拼接的代码一直是处于一种很繁琐的状态，拼接一张表的代码动则几十行，用 Mybatis 就只需要几行就搞定了
>
> 主要内容：
> 1. if 标签
> 2. where 标签
> 3. trim 标签

### 正文

#### 1. if 标签

使用 if 标签来判断参数是否为空，来达到 SQL 拼接的效果：

```
    <select id="selectProduct" resultType="bean.Product">
        SELECT * FROM product
        WHERE id = 1
        <if test="name != null">
            AND name LIKE #{name}
        </if>
    </select>
```

在 if 标签中有一个 test 属性，它用来进行条件判断，如果 test 属性中的语句为 true，就将 if 标签中的语句添加至上面的 SQL 语句中，目前，我都是用在 Web 项目中使用 if 标签配合表单的输入来进行模糊查找

那么如果所有的 if 标签都不成立呢？马上就会说到

#### 2. where 标签

这里给出一条新的 SQL 语句：

```
    <select id="selectProduct" resultType="bean.Product">
        SELECT * FROM product
        WHERE
        <if test="id != null">
            id = #{id}
        </if>
        <if test="name != null">
            AND name LIKE #{name}
        </if>
    </select>
```

如果上述的所有 if 标签都不成立呢，这 SQL 语句就变为：

```
        SELECT * FROM product
        WHERE
```

这时候就不能执行了，因为发生了语法错误，所以我们改用 where 标签：

```
        SELECT * FROM product
        <where>
            <if test="id != null">
                id = #{id}
            </if>
            <if test="name != null">
                AND name LIKE #{name}
            </if>
        </where>
```

where 标签存在的条件是至少有一个子标签存在，也就是至少有一个 if 标签成立，所以如果所有 if 标签都不成立，SQL 语句就变为：

```
        SELECT * FROM product
```

#### 3. trim 标签

我们**假设**，在上面的 SQL 语句中还有一个问题：如果 id 为空，而 name 不为空，那么 SQL 语句就变为：

```
        SELECT * FROM product
        WHERE AND name LIKE #{name}
```

虽然这个假设不成立，在 where 标签和接下来要说的 set 标签中不会出现多余的情况（是否可以认为是内置了 trim 标签），但是我们还是要试一试，多出来一个 AND，看看 trim 标签如何去除：

```
        SELECT * FROM product
        <trim prefix="WHERE" prefixOverrides="AND ">
            <if test="id != null">
                id = #{id}
            </if>
            <if test="name != null">
                AND name LIKE #{name}
            </if>
        </trim>
```

将 where 标签换成 trim 标签，prefix 属性表示我在这个标签前面需要添加什么，我这里添加 WHERE，因为我们已经把 where 标签去掉了，为了语法正确性，添加 WHERE

prefixOverrides 表示，如果指定的内容是多余的（这里指的是 AND以及一个空格），就会去掉

所以，如果 id 为空，name 不为空，SQL 语句就变为：

```
        SELECT * FROM product
        WHERE name LIKE #{name}
```

这时候就能正常运行了

有的人会想，那如果有多个 AND，岂不是就有多个 WHERE？

稍微修改一下：

```
        <trim prefix="WHERE" prefixOverrides="AND ">
            <if test="barcode != null">
                AND barcode LIKE #{barcode}
            </if>
            <if test="name != null">
                AND name LIKE #{name}
            </if>
            <if test="units != null">
                AND units LIKE #{units}
            </if>
        </trim>
```

其实有多个 if 标签，要去掉 AND 的就只有第一个成立的，比如说 barcode 不为空，就用 WHERE 来替换 AND，后面的不管成立不成立，都不会影响语法的正确性

注意，是语法的正确性，不管 name 或 units 是否为空，都不会导致 SQL 语句无法执行，因为 WHERE 后面已经有一条 `barcode LIKE #{barcode}`

#### 4. set 标签

set 标签用于动态更新，也就是更新那些需要更新的行，我们来看看用法：

```
        UPDATE product
        <set>
            <if test="barcode != null">
                barcode = #{barcode},
            </if>
            <if test="name != null">
                name = #{name},
            </if>
            ...
        </set>
```

这里由于篇幅关系，只给出更新两项内容的语句，其余的类似

在 if 标签中的语句中最后又一个逗号，这就和上面说的 AND 是类似的，set 标签会自动去除多余的逗号，否则就会有语法错误，如果你想用 trim 来达到这种效果：

```
        UPDATE product
        <trim prefix="SET" suffixOverrides=",">
            <if test="barcode != null">
                barcode = #{barcode},
            </if>
            <if test="name != null">
                name = #{name},
            </if>
        </trim>
```

这个标签表示，去掉多余的逗号，并在前面加上 SET，使其成为语法正确的语句

总而言之，使用动态 SQL 能够避免 SQL 拼接时可能会发生的种种错误
