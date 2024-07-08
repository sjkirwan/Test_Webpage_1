<script>
    import ExcelJS from 'exceljs';
    import { onMount } from 'svelte';
    let test = {};
    let Onlycurrent = false;
  
      // Initialize an array to store the row objects
      let rowData = [];
    async function handleDownload() {
        const resultWorkbook = new ExcelJS.Workbook();

        // Create a new worksheet
        const resultWorksheet = resultWorkbook.addWorksheet('Result');

        // Define the titles for the columns
        const new_titles = ['Counterparty', 'Accounts Receivable', 'Accounts Payable', 'Positive', 'Negative', 'Future Order Positive', 'Future Order Negative', 'MTM', 'Exposure'];

        // Add the titles to the first row of the worksheet
        resultWorksheet.addRow(new_titles);

        // Iterate over each key-value pair in the test object
        for (const key in test) {
            const row = test[key];
            
            // Create an array with the values for each column
            const values = [
                key,
                row.accountsReceivable,
                row.accountsPayable,
                row.pos,
                row.neg,
                row.futureOrderPos,
                row.futureOrderNeg,
                row.MTM_val,
                row.exposure
                
            ];
            
            // Add the values to a new row in the worksheet
            resultWorksheet.addRow(values);
        }

        // Generate the Excel file
        const buffer = await resultWorkbook.xlsx.writeBuffer();

        // Create a Blob from the buffer
        const blob = new Blob([buffer], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' });

        // Create a download link for the Blob
        const downloadLink = document.createElement('a');
        downloadLink.href = URL.createObjectURL(blob);
        downloadLink.download = 'result.xlsx';
                // ... existing code ...

                // Trigger a click event on the download link to start the download
        console.log('download');
        downloadLink.click();
    }
    function onlyCurrent(){
        Onlycurrent = true;
        //readExcelFile(file);
    }
  
    function argbToRgba(argb) {
        // Convert hex ARGB to separate components
        const alpha = parseInt(argb.slice(0, 2), 16) / 255;
        const red = parseInt(argb.slice(2, 4), 16);
        const green = parseInt(argb.slice(4, 6), 16);
        const blue = parseInt(argb.slice(6, 8), 16);
      
        // Return RGBA string
        return `rgba(${red}, ${green}, ${blue}, ${alpha.toFixed(2)})`;
      }
  
    const readExcelFile = async (file) => {
          rowData = [];
      try {
        const data = new Uint8Array(await file.arrayBuffer());
        const workbook = new ExcelJS.Workbook();
        await workbook.xlsx.load(data);
        const altworkbook = new ExcelJS.Workbook();
        await altworkbook.xlsx.load(data);

        let excelTitles = [];
        let excelData = [];

    // excel to json converter (only the first sheet)
        altworkbook.worksheets[0].eachRow((row, rowNumber) => {
        // rowNumber 0 is empty
            if (rowNumber > 0) {
            // get values from row
                let rowValues = row.values;
            // remove first element (extra without reason)
                rowValues.shift();
            // titles row
                if (rowNumber === 1) excelTitles = rowValues;
            // table data
                else {
                // create object with the titles and the row values (if any)
                    let rowObject = {}
                    for (let i = 0; i < excelTitles.length; i++) {
                        let title = excelTitles[i];
                        let value = rowValues[i] ? rowValues[i] : 0;
                        rowObject[title] = value;
                    }
                    excelData.push(rowObject);
               
                }
            }
        })
        if (Onlycurrent) {
            excelData = excelData.filter(row => {
                const date = new Date(row.date);
                const currentMonth = new Date().getMonth();
                return date.getMonth() === currentMonth;
            });
            
        }
        // Calculate the "neg" key for each row
        excelData.forEach(row => {
            const realizedInvoices = row.realizedInvoices;
            
            if(realizedInvoices == null){
                row.neg = 0;
                row.pos = 0;
            }
            else if (realizedInvoices < 0) {
                row.neg = realizedInvoices;
                row.pos = 0;
            } else if (realizedInvoices > 0){
                row.neg = 0;
                row.pos = realizedInvoices;
            }else{
                row.neg = 0;
                row.pos = 0;
            }     
            
        });


        excelData.forEach(row => {
            if(row.realizedInvoices === 0 && row.forwardNotional > 0 && row.forwardNotional > 0){
                row.futureOrderPos = row.forwardNotional;
                row.futureOrderNeg = 0;
            }
            else if (row.realizedInvoices === 0 && row.forwardNotional > 0 && row.forwardNotional < 0) {
                row.futureOrderPos = 0;
                row.futureOrderNeg = row.forwardNotional;
            } else {
                row.futureOrderPos = 0;
                row.futureOrderNeg = 0;
            }           
        });
        excelData.forEach(row => {
            //const realizedInvoices = row['Realized Invoices'];
            if(row.forwardMTM != 0){
                row.MTM_val = row.forwardPNL;
                console.log(row.MTM_val);
                
            
            } else {
                row.MTM_val = 0;
            }           
        });
        const test = excelData.reduce((acc, curr) => {
            const counterparty = curr.counterparty;
            if (!acc[counterparty]) {
                acc[counterparty] = {
                accountsReceivable: 0,
                accountsPayable: 0,
                pos: 0,
                neg: 0,
                futureOrderPos: 0,
                futureOrderNeg: 0,
                MTM_val: 0
                };
            }
            if (counterparty == 'AVA'){
                console.log(acc[counterparty].accountsReceivable);
            }
            acc[counterparty].accountsReceivable += curr.accountsReceivable;
            acc[counterparty].accountsPayable += curr.accountsPayable;
            acc[counterparty].pos += curr.pos;
            acc[counterparty].neg += curr.neg;
            acc[counterparty].futureOrderPos += curr.futureOrderPos;
            acc[counterparty].futureOrderNeg += curr.futureOrderNeg;
            acc[counterparty].MTM_val += curr.MTM_val;
            return acc;
        }, {});
        
        for (const key in test) {
            const row = test[key];
            test[key]['exposure']= test[key].accountsReceivable+ test[key].accountsPayable+test[key].pos+test[key].neg+test[key].futureOrderPos+test[key].futureOrderNeg + test[key].MTM_val;

            // Perform operations on each row object
            // Example: console.log(row.accountsReceivable);
        }        
        var result = excelData.filter((x)=>x.realizedInvoices === 0 && x.forwardNotional > 0);
        console.log(result);
        console.log(excelData);
        // Create a new workbook
  
        const worksheet = workbook.worksheets[0];
        /*const workbooktest = new ExcelJS.Workbook();
        await workbook.xlsx.load(data);
        const json = JSON.stringify(workbooktest.model);
        console.log(json);*/
  
        
  
        // Get titles from the first row
        const titles = resultWorksheet.getRow(1).values;
        
        // Iterate over each row starting from the second row
        for (let rowIndex = 2; rowIndex <= (resultWorksheet.rowCount); rowIndex++) {
          const rowObject = {cells: [], images: []};
  
          // Iterate over each cell in the row
          for (let colIndex = 1; colIndex <= resultWorksheet.columnCount; colIndex++) {
  
            const cell = resultWorksheet.getCell(rowIndex, colIndex);
            const cellValue = cell.value;
            rowObject.cells[titles[colIndex]] = { value: null, comment: null, color: 'transparent'}
            // Add cell data to the row object
            rowObject.cells[titles[colIndex]].value = cellValue;
  
            // Extract cell color
            if (cell.style?.fill?.fgColor?.argb) {
              rowObject.cells[titles[colIndex]].color = argbToRgba(cell.style.fill.fgColor.argb); // Hex color value
            }
  
            // Extract cell comment
            if (cell._comment?.note?.texts?.length > 1) {
              rowObject.cells[titles[colIndex]].comment = {
                author: cell._comment.note.texts[0].text,
                text: cell._comment.note.texts[1].text
              }
  
            }
  
          }
          // Add the row object to the array
          rowData.push(rowObject);
        }
  
              // Get embedded images and convert to base64
        resultWorksheet.getImages().map((image, index) => {
          const img = workbook.model.media.find(m => m.index === image.imageId);
          if(img != undefined && rowData[image.range.tl.nativeRow - 1] != undefined){
            const base64Image = `data:${img.type};base64,${img.buffer.toString('base64')}`;
            // console.log(image)
            // Log image details, including cell coordinates
            let imageObj = {
              base64Image,
              // filename: extractFileNameFromBase64(base64Image),
              "row": image.range.tl.nativeRow,
              "col": image.range.tl.nativeCol
            }
            // Ensure that .images is an array before pushing
          
            // Push the image object to the row object
            rowData[image.range.tl.nativeRow - 1].images.push(imageObj)
          } else {
            console.log("image not found", index, workbook.model.media, worksheet.getImages())
          }
        });
  
              rowData = rowData;
      } catch (error) {
        console.error('Error reading Excel file:', error.message);
      }
    };
  
      let file;
    const handleFileChange = (event) => {
      file = event.target.files[0];
      readExcelFile(file);
    };
  
  </script>
  
  
  <main>
    <h2 class="app-title">Upload an Excel file containing energy transactions</h2>
    <p class="app-description"></p>
    <div class="file-input-container">
      <label for="fileInput" class="file-input-label">
        <input type="file" id="fileInput" accept=".xls, .xlsx" on:change={handleFileChange} />
        Upload Excel File
      </label>
    </div>
    <button on:click={handleDownload}>Download Excel File</button>
    <button on:click={onlyCurrent}>Only Current</button>
  {rowData.length} Rows
    {#if rowData.length > 0}
      <div class="table-container">
        <table>
          <thead>
            <tr>
              {#each Object.keys(rowData[0].cells) as title}
                <th>{title}</th>
              {/each}
              <th>Images</th>
            </tr>
          </thead>
          <tbody>
            {#each rowData as row}
              <tr>
                {#each Object.values(row.cells) as cell}
                  <td class="cell" style="background: {cell.color};">
                    {cell.value ?? ''}
                    {#if cell.comment}
                      <div class="comment">
                        <small><i><b>{cell.comment?.author}</b> {cell.comment?.text}</i></small>
                      </div>
                    {/if}
                  </td>
                {/each}
                <td class="image-cell">
                  {#if row?.images != undefined && row.images.length > 0}
                    <ul class="image-list">
                      {#each row.images as image}
                        <li>
                          <span class="image-coordinates">{image.row}, {image.col}</span>
                          <img src={image.base64Image} alt={`Image`} />
                        </li>
                      {/each}
                    </ul>
                  {/if}
                </td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
    {/if}
  </main>
  
  <style>
    /* Add your existing styles here */
  
    body {
      font-family: 'Arial', sans-serif;
      background-color: #f8f8f8;
      margin: 0;
      padding: 0;
    }
  
    .app-title {
      text-align: center;
      color: #333;
      margin-top: 20px;
      font-size: 28px;
    }
  
    .app-description {
      text-align: center;
      color: #666;
      margin-bottom: 20px;
    }
  
    .file-input-container {
      text-align: center;
      margin-bottom: 20px;
    }
  
    .file-input-label {
      background-color: #007bff;
      color: #fff;
      padding: 10px 20px;
      border-radius: 5px;
      cursor: pointer;
      font-size: 16px;
    }
  
    .table-container {
      overflow-x: auto;
      margin: 20px;
      background-color: #fff;
      border-radius: 8px;
      box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    }
  
    table {
      width: 100%;
      border-collapse: collapse;
    }
  
    th, td {
      padding: 12px;
      text-align: left;
      border-bottom: 1px solid #eee;
          border-right: 1px solid #eee;
    }
  
    th {
      background-color: #007bff;
      color: #fff;
    }
  
    .cell {
      position: relative;
    }
  
    .comment {
      position: absolute;
      top: 0%;
      left: 0;
      background-color: #fff;
      padding: 8px;
      border: 1px solid #ddd;
      border-radius: 4px;
      box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
      z-index: 1;
    }
  
    .image-cell {
      text-align: center;
      vertical-align: middle;
          
    }
  
    .image-list {
      list-style: none;
      padding: 0;
      margin: 0;
          display: flex;
    }
  
    .image-list li {
      margin-bottom: 20px;
    }
  
    .image-coordinates {
      display: block;
      margin-bottom: 10px;
      color: #666;
    }
  
      img {
          max-height: 50px;
      }
  </style>