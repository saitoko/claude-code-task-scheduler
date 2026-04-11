# Claude Code Task Scheduler Template

Claude Code の Remote Triggers（3枠制限）を最大活用するためのタスクスケジューラー設計テンプレートです。

## 概要

Claude Code の Remote Triggers は現在 **最大3枠**（daily / hourly / adhoc など）しか設定できません。このテンプレートでは、その3枠を「ディスパッチャー」として使い、実際のタスクをスケジュールファイルで管理する設計パターンを提供します。

トリガー枠を消費せずにタスクを追加・無効化できるため、スケジュール管理の柔軟性が大幅に向上します。

## 仕組み

```
Remote Trigger (daily)
  └─ ディスパッチャーが daily.md を読む
       └─ enabled: true のタスクを上から順に実行
            └─ 各タスクの詳細は tasks/{task-id}.md を参照

Remote Trigger (hourly)
  └─ ディスパッチャーが hourly.md を読む
       └─ hours フィールドで実行時間帯をフィルタ
            └─ 該当タスクを実行

Remote Trigger (adhoc)
  └─ ディスパッチャーが adhoc.md を読む
       └─ enabled: true のタスクを実行
            └─ 実行後、enabled を false に戻す
```

## ディレクトリ構成

```
.claude/schedules/
  daily.md          # 毎朝実行タスク一覧
  hourly.md         # 毎時実行タスク一覧
  adhoc.md          # 臨時タスク（手動トリガー用）
  tasks/
    hello-world.md  # サンプルタスク定義
    (your-task).md  # 追加したいタスクをここに配置
logs/
  dispatch/         # タスク実行ログ（YYYY-MM-DD_{task-id}.md）
```

## 使い方

### Step 1: このリポジトリをテンプレートとして使用

GitHub の「Use this template」ボタンから新しいリポジトリを作成します。

### Step 2: タスクを追加する

1. `tasks/` にタスク定義ファイルを作成

```markdown
# my-task

タスクの説明。

1. 手順1
2. 手順2
3. git add → git commit → git push
```

2. `daily.md` または `hourly.md` にタスクを登録

```markdown
## task-id: my-task

- enabled: true
- description: タスクの概要
- condition: always
- task-file: tasks/my-task.md
```

3. コミット & プッシュしてトリガーが次回実行されるのを待つ

### Step 3: Remote Triggers を設定する

Claude Code の [Scheduled Triggers](https://claude.ai/code/scheduled) ページで以下のトリガーを作成します。

| トリガー名 | スケジュール | プロンプト |
|-----------|-------------|-----------|
| Daily Dispatcher | 毎日 06:00 UTC | `cd claude-code-task-scheduler && .claude/schedules/daily.md を読み、enabled: true のタスクを上から順に実行してください。` |
| Hourly Dispatcher | 毎時 | `cd claude-code-task-scheduler && .claude/schedules/hourly.md を読み、enabled: true のタスクを上から順に実行してください（JST 現在時刻を確認し hours フィールドの範囲外であればスキップ）。` |
| Adhoc Dispatcher | 手動 | `cd claude-code-task-scheduler && .claude/schedules/adhoc.md を読み、enabled: true のタスクを実行し、実行後に enabled を false に戻してコミットしてください。` |

> **⚠️ 注意**
>
> - **`cd` は必須です。** Remote Triggers の実行環境はリポジトリルートが作業ディレクトリとは限りません。`cd` を省略するとスケジュールファイルが見つからず、エラーも出ずにサイレントに失敗します。
> - **日付は UTC ではなくローカルタイムゾーンを使ってください。** タスク定義内で日付を扱う場合、`date -u`（UTC）を使うと早朝実行時に1日ズレます。`TZ=Asia/Tokyo date +%Y-%m-%d` のようにローカル TZ を明示してください。

### Step 4: 手動テストで動作確認

初回は必ず [Scheduled Triggers](https://claude.ai/code/scheduled) の **Run now** ボタンで手動テストしてください。プロンプトの `cd` パスやタスクファイルの参照が正しく動作することを確認してから、定期スケジュールを有効にします。

## condition フィールド

スケジュールファイルの各タスクに condition を設定することで、実行日を絞り込めます。

| condition | 説明 | 例 |
|-----------|------|---|
| `always` | 毎回実行 | `condition: always` |
| `weekday: {曜日}` | 特定曜日のみ | `condition: weekday: mon` |
| `monthday: {日}` | 特定日のみ | `condition: monthday: 1` |
| `date-range: {開始}..{終了}` | 期間限定 | `condition: date-range: 2026-04-01..2026-04-30` |

複数条件は現在非対応です。複雑な条件はタスクファイル側に記述してください。

## hours フィールド（毎時タスク用）

`hourly.md` のタスクには `hours` フィールドで実行時間帯を制限できます。指定がない場合は常に実行されます。

```markdown
- hours: 9-18    # 9時〜18時のみ実行（JST）
- hours: 7-22    # 7時〜22時のみ実行（JST）
```

## adhoc の使い方

臨時タスクを手動実行したいときに使います。

1. `adhoc.md` の `enabled` を `true` に変更し、実行内容を記述
2. `git commit & push`
3. [Scheduled Triggers](https://claude.ai/code/scheduled) から adhoc トリガーを手動実行
4. 実行後、ディスパッチャーが自動で `enabled: false` に戻してコミット

## ログの仕組み

各タスクは実行結果を `logs/dispatch/YYYY-MM-DD_{task-id}.md` に保存することを推奨します。ログにより実行履歴の追跡・デバッグが容易になります。

ログの例:

```
logs/dispatch/
  2026-04-11_hello-world.md
  2026-04-11_my-task.md
  2026-04-12_hello-world.md
```

## ライセンス

MIT License — 詳細は [LICENSE](LICENSE) を参照してください。
