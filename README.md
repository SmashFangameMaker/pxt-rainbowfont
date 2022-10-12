# pxt-rainbowfont
A test extension of rainbow font 

function print(img: Image, text: string, x: number, y: number, colors: number[], font?: image.Font) {
    x |= 0
    y |= 0
    if (!font) font = image.font8
    let x0 = x
    let cp = 0
    let mult = font.multiplier ? font.multiplier : 1
    let dataW = Math.idiv(font.charWidth, mult)
    let dataH = Math.idiv(font.charHeight, mult)
    let byteHeight = (dataH + 7) >> 3
    let charSize = byteHeight * dataW
    let dataSize = 2 + charSize
    let fontdata = font.data
    let lastchar = Math.idiv(fontdata.length, dataSize) - 1
    while (cp < text.length) {
        let ch = text.charCodeAt(cp++)
        if (ch == 10) {
            y += font.charHeight + 2
            x = x0
        }

        if (ch < 32)
            continue // skip control chars

        // decompose Korean characters
        let arr = [ch]
        if (44032 <= ch && ch <= 55203) {
            ch -= 44032
            arr = [
                Math.idiv(ch, 588) + 0x1100,
                (Math.idiv(ch, 28) % 21) + 0x1161,
            ]
            ch %= 28
            if (ch)
                arr.push(ch % 28 + 0x11a7)
        }

        for (let cc of arr) {
            let l = 0
            let r = lastchar
            let off = 0

            while (l <= r) {
                let m = l + ((r - l) >> 1);
                let v = fontdata.getNumber(NumberFormat.UInt16LE, m * dataSize)
                if (v == cc) {
                    off = m * dataSize
                    break
                }
                if (v < cc)
                    l = m + 1
                else
                    r = m - 1
            }

            off += 2
            for (let i = 0; i < dataW; ++i) {
                let j = 0
                let mask = 0x01
                let c = fontdata[off++]
                while (j < dataH) {
                    if (mask == 0x100) {
                        c = fontdata[off++]
                        mask = 0x01
                    }
                    if (c & mask)
                        img.fillRect(x, y + j * mult, mult, mult, colors[j] || 1)
                    mask <<= 1
                    j++
                }
                x += mult
            }
        }
    }
}

let f = image.scaledFont(image.font8, 4)
game.onPaint(function () {
    print(screen, "Hello\nworld!", 10, 30, [1, 2, 4, 5, 7, 6, 8], f)
})
