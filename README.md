# PDF 套印（使用JS）
## 使用的套件
1. pdf.js 及 pdf.work.js 處理PDF顯示與定位
  * https://mozilla.github.io/pdf.js/
2. pdf-lib 及 pdf-lib/fontkit 處理PDF套印
  * https://pdf-lib.js.org/
5. downloagjs 處理檔案下載
  * https://www.npmjs.com/package/downloadjs
## 畫面位置與PDF位置
## 真實套印至PDF
1. 載入PDF檔
    * 使用方式
    ```javascript=
    const url = file
    const existingPdfBytes = await fetch(url).then(res => res.arrayBuffer())    
    const pdfDoc = await PDFDocument.load(existingPdfBytes)
    ```
2. PDF存檔
    * 使用方式
    ```javascript=
    // Serialize the PDFDocument to bytes (a Uint8Array)
    const pdfBytes = await pdfDoc.save()
    download(pdfBytes, newFileName, "application/pdf"); //使用downloadjs套件
    ```
3. 套入文字
    * 使用方式
    ```javascript=
    Page.drawText(
        point.desc, //文字內容
        {
            x: x, //文字x座標
            y: y, //文字y座標
            size: 8, //字型大小
            font: customFont, //字型檔
            color: rgb(0.95, 0.1, 0.1), //文字的顏色
        })
    ```
    * 關於中文字型
        * 載入字型檔至PDF中
        ```html=
        <script src="https://unpkg.com/@pdf-lib/fontkit@0.0.4"></script>
        ```
        ```javascript=
        pdfDoc.registerFontkit(fontkit) //將字型元件載入
        const font_url = '/web/font/kaiu.ttf'
        const fontBytes = await fetch(font_url).then(res => res.arrayBuffer())
        const customFont = await pdfDoc.embedFont(fontBytes)
        ```
4. 套入圖片
    * 使用方式
        ```javascript=
        Page.drawImage(
            pngImage, //圖片
            {
                x: x,  //圖片座標x
                y: y + 5, //圖片座標y
                width: pngImage.width * Math.min(widthP, heightP) - 5, //圖片寬度
                height: pngImage.height * Math.min(widthP, heightP) - 5, //圖片高度
            })
        ```
    * 載入圖片的方式
        ```javascript=
        let img_url = '圖片位置'
        const pngImageBytes = await fetch(img1_url).then(res => res.arrayBuffer()) //取得圖片ByteArray
        const pngImage = await pdfDoc.embedPng(pngImageBytes) //載入PDF文件中
        ```
    * 載入多張圖片
        因為forEach如果直接使用async/await會執行順序會怪怪的所以必需改用一般的for來執行
        ```javascript=
        async function asyncForEach(array,callback){
            for(let index=0;index<array.length;i++){
                await callback(array[index],index,array)
            }
        }
        
        let pngImageArray = []
        await asyncForEach(['圖片位置1','圖片位置2','圖片位置3'],async function(data,index,array){
            const pngImageBytes = await fetch(img1_url).then(res => res.arrayBuffer()) //取得圖片ByteArray
            const pngImage = await pdfDoc.embedPng(pngImageBytes) //載入PDF文件中
            pngImageArray.push(pngImage)
        })
        ```
    
## 在畫面上使用滑鼠畫框
* mousedowm事件
* mouseup事件
    * Y座標負值
        * 使用screenY算出dowm與up的差距來計算