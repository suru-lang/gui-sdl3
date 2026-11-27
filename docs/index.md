# 「画面」

The base of a GUI library for Suru. It calls SDL3 and SDL3_ttf through external functions.
Write `「画面」 を 読み込んで 使う` in your file.

Call the SDL3 functions on the first thread of the process (the main thread), because macOS windows need it.
Coordinates start at 0 at the top left and grow to the right and down. The unit is points (the same size on high-resolution screens).

Drawing functions are not visible until `画面表示する`.

**Widgets and apps.** `アプリ実行する` builds an app in the style of Elm. The model is the current value of the app. The draw function takes the model and gives a list of `部品` (widgets), and the update function takes a message and the model and gives the new model. `アプリ模擬する` runs the same without a window, for tests.

The widgets (later ones are drawn on top and get the mouse first):

- `文 と 位置 の ラベル`: a label.
- `文 と 四角 と 知らせ の ボタン`: a button. Pressing the left button on it and releasing on the same button sends 知らせ.
- `値 と 文 と 四角 と 関数 の チェック`: a check box. A press sends the message that `関数` gives for `（値 = 偽）`.
- `量 と 最小 と 最大 と 四角 と 関数 の スライダー`: a slider. Pressing and dragging sends the message that `関数` gives for the number at that place.
- `文 と 四角 と 関数 の 入力欄`: a text field. A press focuses it, and typing or Backspace sends the message that `関数` gives for the new string. Return or Esc removes the focus. Text being converted by the IME is shown too.
- `四角 と 色 の 色四角`: a filled rectangle.

```{toctree}
:maxdepth: 2
:caption: API
:glob:

api/*
```
