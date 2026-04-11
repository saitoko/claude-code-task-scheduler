# Daily Schedule — 毎朝実行タスク一覧

ディスパッチャーがこのファイルを読み、enabled: true のタスクを上から順に実行する。
タスクの実行内容は `tasks/{task-id}.md` を参照すること。

---

## task-id: hello-world

- enabled: true
- description: サンプルタスク — 動作確認用
- condition: always
- task-file: tasks/hello-world.md
