# Phase 3 image fixtures

`asymmetric-rgba.png` and `asymmetric-rgb.jpg` are programmatically generated
3 by 2 pixel test images. They contain no third-party artwork or other licensed
material.

They were generated with ImageMagick 7.1.2-27 using:

```text
convert -size 3x2 xc:none -fill '#ff0000ff' -draw 'point 0,0' -fill '#00ff0080' -draw 'point 1,0' -fill '#0000ffff' -draw 'point 2,0' -fill '#ffff00ff' -draw 'point 0,1' -fill '#ff00ffff' -draw 'point 1,1' -fill '#00ffffff' -draw 'point 2,1' -strip -define png:exclude-chunk=date,time asymmetric-rgba.png
convert -size 3x2 xc:white -fill '#e02020' -draw 'point 0,0' -fill '#20e020' -draw 'point 1,0' -fill '#2020e0' -draw 'point 2,0' -fill '#e0e020' -draw 'point 0,1' -fill '#e020e0' -draw 'point 1,1' -fill '#20e0e0' -draw 'point 2,1' -sampling-factor 1x1 -quality 95 -strip asymmetric-rgb.jpg
```

SHA-256 checksums:

```text
8d77c7c7fd4a9b6b6aa677f2b5bf0af0a3da03bed33be3cfbcc5d3a50f89eb35  asymmetric-rgb.jpg
f1eeb4a7c6afe425ad003957df0c8fe11d6c111c6012f2e2f810fb16ceb9009c  asymmetric-rgba.png
```

`asymmetric-rgb.bmp`, `asymmetric-rgb.gif`, and `asymmetric-rgb.tiff` are
programmatically generated 3 by 2 pixel test images with the same opaque colors
as `asymmetric-rgb.jpg`. They contain no third-party artwork or other licensed
material. They were generated with Pillow 12.3.0 by filling an RGB image with
the six colors `#e02020`, `#20e020`, `#2020e0`, `#e0e020`, `#e020e0`,
`#20e0e0` and saving it in each format.

SHA-256 checksums:

```text
b98b78b351b1511a0b9c5a953334bf2e801f3bf8c6922173d4590613bdb9f9a8  asymmetric-rgb.bmp
86367e4786fe0911e54a3c7567e1cbd1f68557de199b1f72ffcf43b184c27982  asymmetric-rgb.gif
e6b956ad3682993147602bda9ff31c4a6755ebf7cf5b4e0f9015069f240e93ec  asymmetric-rgb.tiff
```

`seal-16bit-boundary.png` is a programmatically generated 6000 by 6000 16-bit
RGBA test image, filled with the single opaque colour `#204060` at 16-bit
precision (every 16-bit channel is the 8-bit value repeated). It contains no
third-party artwork or other licensed material. It was generated with CPython
3.14 in a few lines (a zero filter byte per row, `zlib.compress` at level 9, and
`zlib.crc32` for the PNG chunks), with no image library involved:

```text
python3 - <<'EOF'
import struct, zlib
w = h = 6000
raw = bytearray()
for _ in range(h):
    raw.append(0)
    raw.extend(b'\x20\x20\x40\x40\x60\x60\xff\xff' * w)
chunk = lambda t, d: struct.pack('>I', len(d)) + t + d + struct.pack('>I', zlib.crc32(t + d) & 0xffffffff)
open('seal-16bit-boundary.png', 'wb').write(
    b'\x89PNG\r\n\x1a\n'
    + chunk(b'IHDR', struct.pack('>IIBBBBB', w, h, 16, 6, 0, 0, 0))
    + chunk(b'IDAT', zlib.compress(bytes(raw), 9))
    + chunk(b'IEND', b''))
EOF
```

SHA-256 checksums:

```text
264bfb51e016bb80c612db7dbab8d6070b91e5b4b4b0f8f7f8a2012842496c55  seal-16bit-boundary.png
```
