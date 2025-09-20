---
title: React 概要ガイド（基礎から実務の要点まで）
---

<!-- LP Toast: ランディングに戻る -->
<a href="./index.html" class="lp-toast">← LPに戻る</a>
<style>
.lp-toast{position:fixed;right:16px;bottom:16px;background:#111;color:#fff;padding:10px 14px;border-radius:8px;box-shadow:0 8px 24px rgba(0,0,0,.28);text-decoration:none;font-weight:600;z-index:9999;opacity:.95}
.lp-toast:hover{opacity:1;transform:translateY(-1px)}
</style>

# React 概要ガイド（基礎から実務の要点まで）

このページは、Web フロントエンドで広く使われる React の基本概念と開発の実務ポイントを、プロジェクト文脈（Docker 開発）にあわせてまとめたものです。

## React とは
- UI を「コンポーネント」の組み合わせとして作るライブラリ（Facebook 発）。
- 仮想 DOM により宣言的に UI を記述し、状態変化に応じて差分更新します。
- 単体でも使えるが、実務では Vite/Next.js などのツールと併用が一般的。

## なぜ使うか
- 再利用可能なコンポーネント設計で保守性が高い。
- 豊富なエコシステム（状態管理、ルーティング、デザインシステム、テストなど）。
- TypeScript と相性が良く、大規模開発でも運用しやすい。

---

## コア概念（最低限）

### コンポーネントと JSX（関数コンポーネント）
```tsx
// src/components/Hello.tsx
export function Hello({ name }: { name: string }) {
  return <p>Hello, {name}!</p>; // JSX: HTML風に書けるJavaScript/TypeScript拡張
}
```

### Props（親→子の入力）
- 読み取り専用。子から props を直接変更しない。

### State（useState）と再レンダリング
```tsx
import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount((c) => c + 1)}>
      Count: {count}
    </button>
  );
}
```

### 副作用（useEffect）とクリーンアップ
```tsx
import { useEffect } from 'react';

useEffect(() => {
  const id = setInterval(() => console.log('tick'), 1000);
  return () => clearInterval(id); // アンマウント時に後片付け
}, []); // 依存配列が空=初回のみ
```

### リスト描画と key
```tsx
{items.map((item) => (
  <li key={item.id}>{item.label}</li>
))}
```
- key は安定・一意。index を key に使うのは避ける。

### 条件分岐・イベント
```tsx
{isLoading ? <Spinner /> : <Content />}
<button onClick={(e) => console.log(e.currentTarget)}>Click</button>
```

### フォーム（制御コンポーネント）
```tsx
const [name, setName] = useState('');
<input value={name} onChange={(e) => setName(e.target.value)} />
```

### 状態の持ち上げ（Lifting State Up）
- 兄弟間で共有したいときは、最も近い共通の親に state を持たせる。

### コンテキスト（Context）とカスタムフック
```tsx
// 共有状態は Context + 専用フックで隠蔽
const ThemeContext = createContext<'light' | 'dark'>('light');
export const useTheme = () => useContext(ThemeContext);
```

### メモ化（パフォーマンス）
- `useMemo`（値）/`useCallback`（関数）/`React.memo`（コンポーネント）を状況に応じて利用。
- 早すぎる最適化は逆効果。測ってから適用（DevTools Profiler）。

---

## ルーティングとアプリスキャフォールド
- Vite + React（軽量 SPA）: `react-router-dom` を使用。
- Next.js（推奨・SSR/SSG対応）: ファイルベースルーティング、App Router、サーバーコンポーネント対応。

例: Next.js の最小ページ
```tsx
// app/page.tsx
export default function Page() {
  return <main>Hello Next.js</main>;
}
```

---

## データ取得とサーバー連携
- fetch + `async/await` で基本は十分。
- サーバー状態は `TanStack Query`（React Query）が実務で定番。キャッシュ・再取得・ローディング管理が楽。
- Next.js ではサーバーコンポーネントでのデータ取得も選択肢。

```tsx
// 例: 基本の fetch
async function getUsers() {
  const res = await fetch('/api/users');
  if (!res.ok) throw new Error('Network error');
  return res.json();
}
```

---

## 状態管理の選択肢（使い分け）
- ローカル状態: `useState` / `useReducer`
- アプリ全体の UI 状態: Context + reducer、小規模なら `Zustand`
- サーバー状態（APIデータ）: `TanStack Query`
- 大規模/厳密な制御: `Redux Toolkit`

---

## スタイリング
- CSS Modules / Tailwind CSS / styled-components / MUI など。チーム規約に合わせる。
- コンポーネントの `className` を基準に、UI ライブラリと組み合わせるのが実務では多い。

---

## テスト
- Unit/UI: `Vitest` or `Jest` + `@testing-library/react`
- E2E: `Playwright` or `Cypress`

```tsx
// 例: Testing Library
render(<Counter />);
fireEvent.click(screen.getByRole('button'));
expect(screen.getByText(/Count: 1/)).toBeInTheDocument();
```

---

## よくある落とし穴
- `key` の欠落/不安定な key（index 使用）
- `useEffect` の依存配列不足や無限ループ
- ステートの直接ミューテート（不変性を守る）
- 過剰な再レンダリング（props の参照が毎回変わる等）
- 非同期処理の中断漏れ（アンマウント時のキャンセル）

---

## このプロジェクトでの動かし方（Docker 開発）
- 起動: `docker compose -f infra/compose/compose.dev.yml --profile dev up -d --build`
- Web アプリは `http://localhost:${WEB_PORT}`（デフォルト 3000）
- ソース編集はホスト側の `web/` を更新 → ホットリロード

推奨フォルダ構成（Vite 例）
```
web/
  src/
    components/
    pages/
    hooks/
    styles/
    App.tsx
    main.tsx
  index.html
```

---

## コード規約と品質
- Lint/Format: ESLint + Prettier
- TypeScript: 型ガイドラインに従い、`any` の安易な使用を避ける
- アクセシビリティ: `aria-*` 属性、コントラスト、キーボード操作に配慮

---

## リソース
- React Docs（最新・必読）: https://react.dev/
- React Router: https://reactrouter.com/
- Next.js: https://nextjs.org/
- TanStack Query: https://tanstack.com/query/latest
- Testing Library: https://testing-library.com/

このガイドは最小限の実務指針です。チームの技術選定（Vite/Next、状態管理、スタイル）に合わせ、詳細は各ドキュメントを参照してください。
