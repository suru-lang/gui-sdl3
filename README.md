English | [日本語](README_jp.md)

# gui-sdl3

「画面」 ("screen"), a GUI library for [Suru (「する」)](https://github.com/suru-lang/suru) that opens windows and draws shapes and Japanese text. It calls [SDL3](https://github.com/libsdl-org/SDL) and [SDL3_ttf](https://github.com/libsdl-org/SDL_ttf) as foreign functions. It runs on macOS and Linux (Windows once Suru's runtime library supports it).

There is a base layer of windows, drawing, text and events, and on top of it widgets written in Suru (labels, buttons, checkboxes, sliders, text fields). Apps using widgets are written like Elm, with a "model", a "view" and an "update".

```
「画面」 を 読み込んで 使う

始める とは 以下の手順です
  w とは 「こんにちは」 を 320 で 120 に 窓作成した 結果 です
  w を （255 と 255 と 255 の 色） で 全面塗装します
  w に （20 と 20 と 40 と 40 の 四角） を （220 と 80 と 60 の 色） で 四角塗装します
  w に 「こんにちは、世界」 を （80 と 28） で （0 と 0 と 0 の 色） から 文字描画します
  w を 画面表示します
  3000 を 一時停止します
  w を 窓閉鎖します
  以上です
始める
```

An app using widgets is written as follows. The whole thing is [例/部品見本.suru](例/部品見本.suru).

```
「画面」 を 読み込んで 使う

合図 とは 増加 か 減少 です

合図 を 数 で 数更新する は 数 を 返す
増加 を n で 数更新する とは n + 1 です
減少 を n で 数更新する とは n - 1 です

数 を 数描画する は 部品 の リスト を 返す
n を 数描画する とは （（n を 文字列化する） と （20 と 20） の ラベル） と （「+1」 と （20 と 50 と 80 と 36 の 四角） と 増加 の ボタン） と （「−1」 と （110 と 50 と 80 と 36 の 四角） と 減少 の ボタン） です

「数える」 を 240 で 120 に 0 から 数更新すること へ 数描画すること より アプリ実行します
```

An example that places squares on click (reading events directly, without widgets) is [例/四角置き.suru](例/四角置き.suru).

## Installing

1. Install SDL3 and SDL3_ttf (3.2 or later).
   - macOS: `brew install sdl3 sdl3_ttf`
   - Debian, Ubuntu (25.04 or later): `sudo apt install libsdl3-dev libsdl3-ttf-dev`
   - Fedora: `sudo dnf install SDL3-devel SDL3_ttf-devel`
   - Windows (MSYS2 UCRT64): `pacman -S mingw-w64-ucrt-x86_64-sdl3 mingw-w64-ucrt-x86_64-sdl3-ttf`. Use the UCRT64 shell with clang, lld and pkgconf (`pacman -S mingw-w64-ucrt-x86_64-clang mingw-w64-ucrt-x86_64-lld mingw-w64-ucrt-x86_64-pkgconf`); the SDL3 DLLs are found on the UCRT64 `PATH`.
   - Where there are no packages, such as Ubuntu 24.04, install from source (the steps in `.github/workflows/試験.yml`).
2. Install Suru 0.9.1 or later (the library writes its C functions with 「以下は外部の入出力動作です … 以上です」) and kumu. This also covers Windows, which needs 0.8.0 or later.
3. In your project, run `kumu 追加 github.com/suru-lang/gui-sdl3`.

## Usage

The list of what is available is at the head of [画面.suru](画面.suru).

- Open a window with `題 を 幅 で 高さ に 窓作成する`, draw, then show it with `画面表示する`. Redraw from `全面塗装する` every time (it waits for the screen refresh, so looping doesn't use up the CPU).
- `窓事象取得する` returns all pending events and doesn't wait. Loops are written by calling yourself at the end of a procedure (see the examples).
- Colors are `赤 と 緑 と 青 の 色` (red, green, blue, 0 to 255), rectangles are `左 と 上 と 幅 と 高さ の 四角` (left, top, width, height), and points are lists `横 と 縦` (x and y).
- Text uses the OS's Japanese font (Hiragino on macOS, Yu Gothic or Meiryo on Windows, Noto Sans CJK or IPA Gothic on Linux) at size 18. `書体設定する` changes it.

## Decisions

### Base

1. **SDL3 functions are called on the main thread.** macOS windows can only be operated from the first thread, so the library writes `外部の「SDL3」 を 主スレッドで 読み込む`. Drawing from code running concurrently is fine (in compiled programs; the interpreter only from the first thread).
2. **Structs are assembled into bytes in Suru.** SDL rectangles (`SDL_FRect`) are laid out as four floats, and events (`SDL_Event`) are read from 128 bytes. Float conversion is written in Suru too.
3. **Text is drawn with SDL3_ttf's text engine.** It caches the shapes of drawn glyphs, so redrawing every frame stays fast.
4. **Drawing is in points.** Coordinates, sizes, mouse positions and the window size are all in points. On high-resolution screens (Retina and the like), rectangles and lines are scaled to pixels by the renderer scale, and text is drawn in pixels with a font of the scaled size so it stays crisp. The scale is taken from the screen when the window is created.
5. **Tests run without a screen.** On Linux with `SDL_VIDEO_DRIVER=offscreen`, on macOS with a real window (offscreen can't open OpenGL), both drawing with `SDL_RENDER_DRIVER=software`, and checking the pixels saved as BMP with `画像保存する`.

### Widgets and apps

1. **Apps are written like Elm.** Both the "view", which makes a list of widgets from the model (the app's current value), and the "update", which changes the model from messages sent by widgets, are pure functions. The library takes care of the window, the loop, hit testing and focus. Suru values never change, so this form is the most natural to write.
2. **Widgets are shapes of one type, 部品.** The view just returns a list of them; drawing and dispatching events are the library's job. Later widgets are drawn on top and receive the mouse first.
3. **Checkboxes, sliders and text fields take a function that turns the new value into a message.** The widget type's fields have different names per shape (`切替`, `量変更`, `文変更`), because in a type with alternatives, fields with the same name end up with the same type.
4. **A button sends its message when the left button is pressed on it and released over the same button.** Pressing outside and releasing inside, or using the right button, sends nothing.
5. **Pressing a text field focuses it and starts text input.** Return, Esc or pressing elsewhere unfocuses it. Text being composed by the IME is shown in blue with an underline, and Backspace deletes a character only when not composing.
6. **`アプリ模擬する` lets you test without a window.** It is a pure function returning the model after applying a list of events in order. You can build mouse-down events and the like yourself and pass them.

## Future work

- Horizontal and vertical layout (deciding widget rectangles automatically), scrolling, multi-line text fields, cursor movement inside text fields
- Messages for things that move with time (animation, clocks)
- Re-reading the scale when the window moves to a screen with a different scale
- Loading and drawing image files

## License

MIT. SDL3 and SDL3_ttf are under the zlib license; FreeType, used by SDL3_ttf, is under the FreeType License (FTL) or GPLv2.
