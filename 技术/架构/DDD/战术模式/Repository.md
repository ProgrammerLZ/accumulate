1. 一个聚合对应一个Repository
2. 不要在Repository当中返回DTO，而是返回Entity，在Assembler中做转换逻辑
3. 要注意更新数据库的时候，大量的无用更新操作，要考量是否应该避免