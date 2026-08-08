# Rake Task、`lib/` 与 Zeitwerk 自动加载

> 起因：rake task 是不是都写在 `lib` 里？主流做法是什么？Zeitwerk 默认不是不自动加载 `lib` 吗？Rails 是在弱化 `lib` 的功能吗？

## 1. `.rake` 文件放哪

- Rake task 的**定义文件**（`.rake`）默认放 `lib/tasks/`，Rails 会自动 glob `lib/tasks/**/*.rake` 并加载。
- 加载方式是 `Kernel#load`（执行文件里的 task DSL），**不是** Zeitwerk 的 autoload。
- 所以 "`lib/tasks/*.rake` 能被加载" 和 "Zeitwerk 不 autoload `lib`" 互不矛盾——两条完全不同的机制。

```ruby
# lib/tasks/cleanup.rake
namespace :cleanup do
  desc "清理过期的会话"
  task expired_sessions: :environment do
    Session.where("expires_at < ?", Time.current).delete_all
  end
end
```

## 2. 为什么 `.rake` 不受 Zeitwerk 影响

- Zeitwerk 只关心 `.rb` 文件里定义的**常量**（class/module），按 "常量名 → 文件路径" 规则做 autoload。
- `.rake` 后缀不在 Zeitwerk 扫描范围内，`.rake` 里也不定义常量，因此天然绕开。
- 从 Rails 6.1 / 7 起，`lib` 默认**不在** autoload paths；要自动加载 `lib` 里的 `.rb` 需显式 `config.autoload_lib`。

## 3. 主流做法：task 保持"薄"

`.rake` 里只做参数解析和调度，业务逻辑抽到可测试、可复用的类。

反模式（❌ 逻辑全塞进 task，无法单测/复用）：

```ruby
task generate_reports: :environment do
  # 几十上百行业务逻辑...
end
```

推荐（✅ task 只调一行）：

```ruby
# lib/tasks/reports.rake
namespace :reports do
  desc "生成月度报表"
  task :generate, [:month] => :environment do |_t, args|
    Reports::MonthlyGenerator.new(month: args[:month]).call
  end
end
```

逻辑类放哪：

| 逻辑类型 | 放置位置 | autoload |
|---------|---------|----------|
| 领域/业务逻辑 | `app/services`、`app/models`、`app/lib` 等 | ✅ Zeitwerk 自动加载 |
| 通用工具/与框架无关的库 | `lib/` | 需 `config.autoload_lib` 显式开启 |

如果确实要 autoload `lib/` 里的类（Rails 7+）：

```ruby
# config/application.rb
config.autoload_lib(ignore: %w[assets tasks])
```

`ignore: %w[tasks]` 关键——`lib/tasks` 是 `.rake` 而非常量文件，必须排除，否则 Zeitwerk 找不到对应常量会报错。

## 4. Rails 是在"弱化 `lib`"吗？

**是趋势，但更准确的说法是"重新定位 `lib`"，而不是废弃。**

- **历史**：老 Rails（≤5）默认把 `lib` 放进 `autoload_paths`，大家习惯把各种业务类丢 `lib`。但这带来加载顺序、重载（reload）、常量歧义等一堆坑。
- **转折**：Rails 6 引入 Zeitwerk，Rails 6.1/7 起**默认把 `lib` 移出 autoload/eager-load paths**。官方立场：`lib` 不该默认自动加载。
- **新定位**：
  - **业务逻辑 → `app/` 下**（`app/services`、`app/lib`、`app/jobs`、`app/models`…）。放 `app/` 天然被 Zeitwerk autoload + 开发环境可 reload，无需配置，也最好测。
  - **`lib/` 留给**：rake tasks、与应用领域无关的通用/独立库代码、脚手架/一次性脚本。这类代码往往**不需要自动加载、也不需要 reload**。
- 所以主流是：**"逻辑上移到 `app/`，`lib/` 回归'放独立库和任务'的本分"**。不是砍功能，而是把"自动加载一切"的默认行为收紧，交还给显式配置（`autoload_lib`）。

## 5. 一句话总结

1. `.rake` 放 `lib/tasks/`，由 `load` 加载，和 Zeitwerk 无关。
2. task 保持轻薄，逻辑抽成类，首选放 `app/`（自动 autoload + reload）。
3. 放 `lib/` 的类要 autoload 得显式 `config.autoload_lib(ignore: %w[tasks ...])`。
4. Rails 确实在收紧 `lib` 的默认自动加载，本质是"业务逻辑上移到 `app/`，`lib/` 回归独立库/任务"的重新定位，而非弃用。
