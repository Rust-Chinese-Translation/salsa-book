# 按需（惰性）输入

如果你可以轻松地预先提供所有输入，则 Salsa 输入查询工作得最好。然而，有时输入集是事先不知道的。

一个典型的例子是从磁盘读取文件。虽然在 Salsa 输入查询中可以急切地扫描特定目录并创建内存中的文件树，但更直接的方法是延迟读取文件。

也就是说，当某人第一次请求文件的文本时：

1. 从磁盘读取文件并缓存它
2. 为此路径设置文件系统监视器 (file-system watcher)
3. 在监视器发送更改通知后，使缓存的文件无效

这可以在 Salsa 中使用派生查询以及 `report_synthetic_read` 和 `invalidate` 查询来实现。大致设置如下：

```rust,ignore
#[salsa::query_group(VfsDatabaseStorage)]
trait VfsDatabase: salsa::Database + FileWatcher {
    fn read(&self, path: PathBuf) -> String;
}

trait FileWatcher {
    fn watch(&self, path: &Path);
    fn did_change_file(&mut self, path: &Path);
}

fn read(db: &dyn VfsDatabase, path: PathBuf) -> String {
    db.salsa_runtime()
        .report_synthetic_read(salsa::Durability::LOW);
    db.watch(&path);
    std::fs::read_to_string(&path).unwrap_or_default()
}

#[salsa::database(VfsDatabaseStorage)]
struct MyDatabase { ... }

impl FileWatcher for MyDatabase {
    fn watch(&self, path: &Path) { ... }
    fn did_change_file(&mut self, path: &Path) {
        ReadQuery.in_db_mut(self).invalidate(path);
    }
}
```

* 将查询声明为派生查询（这是默认）
* 在查询实现中，不调用任何其他查询，只是直接从磁盘读取文件
* 因为查询不读取任何输入，所以默认情况下将分配一个 `HIGH` 持久性，我们使用 `report_synthetic_read` 覆盖它
* 查询的结果被缓存，我们必须调用 `invalidate` 来清除这个缓存

这个 [repo] 提供了一个完整的、可运行的文件监视示例，以及关于代码和它正在做的更多解释的[评述][write-up]。

[repo]: https://github.com/ChristopherBiscardi/salsa-file-watch-example/blob/f968dc8ea13a90373f91d962f173de3fe6ae24cd/main.rs
[write-up]: https://www.christopherbiscardi.com/on-demand-lazy-inputs-for-incremental-computation-in-salsa-with-file-watching-powered-by-notify-in-rust

