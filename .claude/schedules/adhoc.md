# Adhoc Schedule — 臨時タスク

手動トリガーで実行する。実行後、enabled: true のタスクは自動で false に戻される。

使い方:
1. 実行したいタスクの enabled を true に変更し、実行内容を記述
2. git commit & push
3. https://claude.ai/code/scheduled から adhoc トリガーを手動実行

---

## task-id: adhoc-template

- enabled: false
- description: （タスクの説明をここに書く）
- condition: always
- agent: （担当エージェント名）

### 実行内容

（具体的な手順をここに書く）
