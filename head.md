# Felix Liu

## 知识

### MongoDB aggregate pipeline 注意事项

- `$limit` 和 `$skip` 位置会影响最终结果：
    - 如果 `$limit` 在前 `$skip` 在后，表示先取 `limit` 条记录，然后跳过前 `skip` 条数记录。
    - 如果 `$skip` 在前 `$limit` 在后，表示先跳过 `skip` 条记录，然后取 `limit` 条数记录。
- `$sort` 操作符要放到 `$limit` 和 `$skip` 前面，在内存中只会维护 `limit` 个数量的文档，不需要将所有的文档维护在内存中，大大降低内存中 `sort` 的压力。
- 在不建立索引的情况下，使用 aggregate 方式查询，其返回结果是不确定的。这个字段建立索引或者增加能确定顺序的排序条件。
- 对于管道中的索引，也很容易出现意外，只有在管道最开始时的 match sort 可以使用到索引，一旦发生过 project 投射，group 分组，lookup 表关联，unwind 打散等操作后，就完全无法使用索引
- 每个阶段管道限制为 100MB 的内存。如果一个节点管道超过这个极限，MongoDB 将产生一个错误。为了能够在处理大型数据集，可以设置 allowDiskUse 为 true 来在聚合管道节点把数据写入临时文件。这样就可以解决 100MB 的内存的限制。
