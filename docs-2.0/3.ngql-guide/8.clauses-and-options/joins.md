# INNER JOIN

`INNER JOIN`是一种连接查询，用于将两个表关联在一起，只返回在两个表中都存在的行。在 NebulaGraph 中，`INNER JOIN`子句可以用来在两个表格之间进行连接查询，以获取更加复杂的查询结果。

## openCypher 兼容性

`INNER JOIN`子句仅适用于原生 nGQL 语法。

## 语法

```ngql
YIELD <column_name_list>
FROM <first_table> INNER JOIN <second_table> ON <join_condition>
```

## 使用说明

- 必须使用`YIELD`子句指定返回的列，并且`YIELD`子句放置在`INNER JOIN`子句之前。
- `INNER JOIN`子句必须包含`ON`子句，`ON`子句指定了连接条件。连接条件只支持等值连接（即`==`）。
- `<first_table>`和`<second_table>`是要连接的两个表，两表名不能相同。
- 可以使用自定义变量来指定表格名。详情参见[使用自定义变量](../3.ngql-guide/4.variable-and-composite-queries/2.user-defined-variables.md)。

## 使用示例

示例一：结合`LOOKUP`和`GO`子句，查找名为`Tony Parker`的球员和他的朋友之间的`like`关系。

1. 先查找名为`Tony Parker`的球员的ID。
2. 再查找名为`Tony Parker`和`Manu Ginobili`的球员之间是否存在`like`关系，如果存在，则将两个球员的 ID 和`like`关系返回。
3. 最后的将两次查询的结果合并在一起，以便获取最终的结果。

```ngql
nebula> $a = LOOKUP ON player WHERE player.name == 'Tony Parker' YIELD id(vertex) as dst, vertex AS v; \
        $b = GO FROM 'Tony Parker', 'Manu Ginobili' OVER like YIELD id($^) as src, id($$) as vid, edge AS e2; \
        YIELD $b.vid AS vid, $a.v AS v, $b.e2 AS e2 FROM $a INNER JOIN $b ON $a.dst == $b.src;
```

示例二：结合`LOOKUP`和`FETCH PROP`子句，查找名为`Tony Parker`的球员和他的朋友之间的`like`关系。

1. 通过以下查询在'player'表中查找名为'Tony Parker'的球员：LOOKUP ON player WHERE player.name == 'Tony Parker' YIELD id(vertex) as src, vertex AS v。该查询将返回一个包含Tony Parker的顶点ID（src）和该顶点的所有属性（v）的结果。
2. 使用以下查询在'like'边缘上查找名为'Tony Parker'的球员和名为'Tim Duncan'的球员之间的关系：FETCH PROP ON like 'Tony Parker'->'Tim Duncan' YIELD src(edge) as src, edge as e。该查询将返回一个包含如上边缘的起始点(src)和边缘本身的属性(e)的结果。
3. 使用 INNER JOIN 子句将第一步和第二步的查询结果联接在一起，以便检索统一结果，包括源点 （src）、源点的属性（v）和找到的like边缘（e）。YIELD $a.src AS src, $a.v AS v, $b.e AS e FROM $a INNER JOIN $b ON $a.src == $b.src。最终的结果是一个包含查询结果的表，其中包括来自两个查询的所有变量（src、v和e）。

```ngql      
nebula> $a = LOOKUP ON player WHERE player.name == 'Tony Parker' YIELD id(vertex) as src, vertex AS v; \
        $b = FETCH PROP ON like 'Tony Parker'->'Tim Duncan' YIELD src(edge) as src, edge as e; \
        YIELD $a.src AS src, $a.v AS v, $b.e AS e FROM $a INNER JOIN $b ON $a.src == $b.src;
```

示例三：结合`LOOKUP`、`GO`和`FIND PATH`子句，查找名为`Tony Parker`的球员和他的朋友之间的`like`关系。

1. 通过名为'Tony Parker'的球员的名称查找他的ID和属性。
2. 从刚才查找到的球员ID开始，向like关系方向前进2到5步，并查找所有满足条件的目标顶点（满足条件的目标顶点的年龄大于30）。
3. 查找所有从a到b之间的路径，并在查询结果中列出路径和目标终点。
4. 查找从b到a的最短路径，并在查询结果中列出路径和起始点。
5. 将第三步和第四步的结果合并，并从两个结果中选择所需的列作为最终结果。具体地，将第三步中找到的从A到B的路径存储在变量$c.forwardPath中，将第二步找到的目标节点的ID存储在变量$c.end中，将第四步中的从B到A的路径存储在变量$d.backwordPath中。最后，从$c和$d中选择需要的列列出最终结果。

```ngql
nebula> $a = LOOKUP ON player WHERE player.name == 'Tony Parker' YIELD id(vertex) as src, vertex AS v; \
        $b = GO 2 TO 5 STEPS FROM $a.src OVER like WHERE $$.player.age > 30 YIELD id($$) AS dst; \
        $c = (FIND ALL PATH FROM $a.src TO $b.dst OVER like YIELD path AS p | YIELD $-.p AS forward, id(endNode($-.p)) AS dst); \
        $d = (FIND SHORTEST PATH FROM $c.dst TO $a.src OVER like YIELD path AS p | YIELD $-.p AS p, id(startNode($-.p)) AS src); \
        YIELD $c.forward AS forwardPath, $c.dst AS end, $d.p AS backwordPath  FROM $c INNER JOIN $d ON $c.dst == $d.src;
```
      