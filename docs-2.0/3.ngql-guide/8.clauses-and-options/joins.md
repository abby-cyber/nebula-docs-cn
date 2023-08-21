# INNER JOIN

`INNER JOIN`是一种连接查询，用于将两个表关联在一起，只返回在两个表中都存在的行。在{{nebula.name}}中，`INNER JOIN`子句可以用来在两个表之间进行连接查询，以获取更加复杂的查询结果。

## openCypher 兼容性

`INNER JOIN`子句仅适用于原生 nGQL 语法。

## 语法

```ngql
YIELD <column_name_list>
FROM <first_table> INNER JOIN <second_table> ON <join_condition>
```

## 使用说明

- 必须使用`YIELD`子句指定返回的列，并且`YIELD`子句需放置在`INNER JOIN`子句之前。
- `INNER JOIN`子句必须包含`ON`子句，`ON`子句指定了连接条件。连接条件只支持等值连接（即`==`）。
- `<first_table>`和`<second_table>`是要连接的两个表，两表名不能相同。
- 可以使用自定义变量来指定表格名。详情参见[使用自定义变量](../3.ngql-guide/4.variable-and-composite-queries/2.user-defined-variables.md)。

## 使用示例

示例一：结合`LOOKUP`和`GO`子句，使用`INNER JOIN`获取`Tony Parker`的球员和他的朋友之间关系的信息。

```ngql
nebula> $a = LOOKUP ON player WHERE player.name == 'Tony Parker' YIELD id(vertex) as dst, vertex AS v; \
        $b = GO FROM 'Tony Parker', 'Manu Ginobili' OVER like YIELD id($^) as src, id($$) as vid, edge AS e2; \
        YIELD $b.vid AS vid, $a.v AS v, $b.e2 AS e2 FROM $a INNER JOIN $b ON $a.dst == $b.src;
```

示例二：结合`LOOKUP`和`FETCH PROP`子句，使用`INNER JOIN`获取`Tony Parker`和`Tim Duncan`之间的关系的信息。

```ngql      
nebula> $a = LOOKUP ON player WHERE player.name == 'Tony Parker' YIELD id(vertex) as src, vertex AS v; \
        $b = FETCH PROP ON like 'Tony Parker'->'Tim Duncan' YIELD src(edge) as src, edge as e; \
        YIELD $a.src AS src, $a.v AS v, $b.e AS e FROM $a INNER JOIN $b ON $a.src == $b.src;
```

示例三：结合`LOOKUP`、`GO`和`FIND PATH`子句，查找和`Tony Parker`之间有特定关系的其他球员，以及这些球员和`Tony Parker`之间的最短路径。

```ngql
nebula> $a = LOOKUP ON player WHERE player.name == 'Tony Parker' YIELD id(vertex) as src, vertex AS v; \
        $b = GO 2 TO 5 STEPS FROM $a.src OVER like WHERE $$.player.age > 30 YIELD id($$) AS dst; \
        $c = (FIND ALL PATH FROM $a.src TO $b.dst OVER like YIELD path AS p | YIELD $-.p AS forward, id(endNode($-.p)) AS dst); \
        $d = (FIND SHORTEST PATH FROM $c.dst TO $a.src OVER like YIELD path AS p | YIELD $-.p AS p, id(startNode($-.p)) AS src); \
        YIELD $c.forward AS forwardPath, $c.dst AS end, $d.p AS backwordPath FROM $c INNER JOIN $d ON $c.dst == $d.src;
```
      