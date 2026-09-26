**LUCI is a simple and lightweight graphics library for C.**

## Features
- Window management
- 2D drawing
- Text rendering
- Input handling

## basic example
```C
int main()
{
    LUCI_init();
    while (!LUCI_window_should_close()) {
        LUCI_begin_drawing();
        LUCI_draw_rectangle(32,32,64,64,LUCI_COLOR_BLUE);
        LUCI_end_drawing();
    }
    return 0;
}
```
## Installation
```bash
git clone https://github.com/WibuKa/luci.git
cd luci
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release                                                                                                                                       147 ↵
cmake --build build
```
