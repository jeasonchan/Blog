

帮妹子用VBA实现了Excel的一段宏，功能是一些插入、删除、查找等等

```VBA
Sub main()
'
' test2 Macro
'

total_row = ActiveSheet.UsedRange.Rows.Count

Dim row_number

For row_number = 2 To total_row

cell_value = CSng(Cells(row_number, "G"))

If cell_value < 0 Then

Cells(row_number, "G").EntireRow.Delete

row_number = row_number - 1

End If

Next


'break vendor code and vendor name

    Columns("G:G").Select
    Selection.Insert Shift:=xlToRight, CopyOrigin:=xlFormatFromLeftOrAbove
Application.DisplayAlerts = False 'disable windows alert

    Columns("F:F").Select
    Selection.TextToColumns Destination:=Range("F1"), DataType:=xlFixedWidth, _
        FieldInfo:=Array(Array(0, 1), Array(9, 1)), TrailingMinusNumbers:=True
    Range("F1").Select
    ActiveCell.FormulaR1C1 = "Vendor Code"
    Range("G1").Select
    ActiveCell.FormulaR1C1 = "Vendor Name"
Application.DisplayAlerts = True 'enable windows alert
 
'insert 2 columns after vendor

    Columns("H:H").Select
    Selection.Insert Shift:=xlToRight, CopyOrigin:=xlFormatFromLeftOrAbove
    Selection.Insert Shift:=xlToRight, CopyOrigin:=xlFormatFromLeftOrAbove
   
'insert 3 columns after plant

    Columns("L:L").Select
    Selection.Insert Shift:=xlToRight, CopyOrigin:=xlFormatFromLeftOrAbove
    Selection.Insert Shift:=xlToRight, CopyOrigin:=xlFormatFromLeftOrAbove

    Selection.Insert Shift:=xlToRight, CopyOrigin:=xlFormatFromLeftOrAbove
   
    'rename columns
   
    Range("H1").Select
    ActiveCell.FormulaR1C1 = "Vendor Country"
    Range("I1").Select
    ActiveCell.FormulaR1C1 = "Category"
    Range("L1").Select
    ActiveCell.FormulaR1C1 = "Plant Name"
    Range("M1").Select
    ActiveCell.FormulaR1C1 = "Segment"
    Range("N1").Select
    ActiveCell.FormulaR1C1 = "Plant Region"
   
    'vlookup values
   
    Range("H2").Select
    ActiveCell.FormulaR1C1 = _
        "=VLOOKUP(RC[-2],'[vendor-category&region.xlsx]Supplier Management Model'!C2:C5,4,0)"
    Range("I2").Select
    ActiveCell.FormulaR1C1 = _
        "=VLOOKUP(RC[-3],'[vendor-category&region.xlsx]Supplier Management Model'!C2:C16,15,0)"
    Range("L2").Select
    ActiveCell.FormulaR1C1 = _
        "=VLOOKUP(RC[-1],'[plant&vendor list.xlsx]Sheet1'!C1:C3,3,0)"
    Range("M2").Select
    ActiveCell.FormulaR1C1 = _
        "=VLOOKUP(RC[-2],'[plant&vendor list.xlsx]Sheet1'!C1:C8,8,0)"
    Range("N2").Select
    ActiveCell.FormulaR1C1 = _
        "=VLOOKUP(RC[-3],'[plant&vendor list.xlsx]Sheet1'!C1:C6,6,0)"
   
    'auotofill
   
   
   
    Range("H2").Select
    Selection.AutoFill Destination:=Range("H2:H" & total_row)
    Range("H2:H" & total_row).Select
    Range("I2").Select
    Selection.AutoFill Destination:=Range("I2:I" & total_row)
    Range("I2:I" & total_row).Select
    Range("L2").Select
    Selection.AutoFill Destination:=Range("L2:L" & total_row)
    Range("L2:L" & total_row).Select
    Range("M2").Select
    Selection.AutoFill Destination:=Range("M2:M" & total_row)
    Range("M2:M" & total_row).Select
    Range("N2").Select
    Selection.AutoFill Destination:=Range("N2:N" & total_row)
    Range("N2:N" & total_row).Select

'insert columns after currency

    Columns("Y:Y").Select
    Selection.Insert Shift:=xlToRight, CopyOrigin:=xlFormatFromLeftOrAbove
    Range("Y1").Select
    ActiveCell.FormulaR1C1 = "Exchange rate"
    Columns("Z:Z").Select
    Selection.Insert Shift:=xlToRight, CopyOrigin:=xlFormatFromLeftOrAbove
    Range("Z1").Select
    ActiveCell.FormulaR1C1 = "USD Price"
    Range("Y2").Select
    ActiveCell.FormulaR1C1 = _
        "=VLOOKUP(RC[-1],'[2020 FX Rates_YTD.xlsx]2020'!C1:C22,22,0)"
    Range("Z2").Select
    Application.CutCopyMode = False
    ActiveCell.FormulaR1C1 = "=RC[-1]*RC[-11]"
    Range("Y2").Select
    Selection.AutoFill Destination:=Range("Y2:Y" & total_row)
    Range("Y2:Y" & total_row).Select
    Range("Z2").Select
    Selection.AutoFill Destination:=Range("Z2:Z" & total_row)
    Range("Z2:Z" & total_row).Select
   
   



End Sub
```
