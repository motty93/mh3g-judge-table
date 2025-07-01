## 砂原採取マクロ

```
Rem Attribute VBA_ModuleType=VBADocumentModule
Option VBASupport 1
Sub allkensaku()
    
    Application.Calculation = xlManual
    
    Dim b As Integer, g As Integer, i As Integer, j As Integer, k As Integer, n As Integer
    Dim sidou As String, sido As String, siido As String, s As Integer, kotei As Long
    
    '報酬読み込み
    Dim housyu(2, 1) As Variant
    Dim hanni(2, 1) As Long
    
    For i = 0 To 2 Step 1
        housyu(i, 0) = Worksheets("Sheet1").Cells(3 + i, 1).Value
        housyu(i, 1) = Worksheets("Sheet1").Cells(3 + i, 2).Value
        
        If i = 0 Then
            hanni(i, 0) = 0
            hanni(i, 1) = housyu(i, 1) - 1
        Else
            hanni(i, 0) = hanni(i - 1, 1) + 1
            hanni(i, 1) = hanni(i - 1, 1) + housyu(i, 1)
        End If
        
    Next i
    
    '入力報酬枠読み込み
    Dim gyou As Long, waku As Integer
    
    '枠判定数
    waku = 7
    
    '固定消費値
    kotei = Worksheets("Sheet1").Cells(12, 2).Value
    
    Dim pata As Integer
    pata = Worksheets("Sheet1").Cells(10, 5).Value
    
    '報酬パターンの読み込み
    Dim nyuuryoku As Variant, seed As Variant, maxl(16) As Long, kekka(16) As String, wari As Long
    nyuuryoku = Worksheets("Sheet1").Range(Worksheets("Sheet1").Cells(3, 5), Worksheets("Sheet1").Cells(3 + pata - 1, 11))
    
    'スキルの読み込み
    Dim skill As Variant, lack As Integer
    skill = Worksheets("Sheet1").Range(Worksheets("Sheet1").Cells(3, 12), Worksheets("Sheet1").Cells(3 + pata - 1, 12))
    
    '入力データを報酬テーブルの範囲データに変換して照合
    Dim nyuuryokuhanni() As Long, furagu As Long
    Dim zouka As Long, mawari As Long, kai As Long, x As Long, y As Long
    
    seed = Worksheets("Sheet2").Range("A2:Q5401")
    
    For b = 0 To 16 Step 1
        maxl(b) = WorksheetFunction.CountA(Worksheets("Sheet2").Range(Worksheets("Sheet2").Cells(1, b + 1), Worksheets("Sheet2").Cells(UBound(seed, 1), b + 1)))
    Next b
    
    ReDim nyuuryokuhanni(waku - 1, 1, pata - 1)
    
    '報酬結果を範囲データに変換して格納
    For k = 1 To pata Step 1
        For g = 1 To waku Step 1
            For j = 0 To 2 Step 1
                If nyuuryoku(k, g) = housyu(j, 0) Then
                    nyuuryokuhanni(g - 1, 0, k - 1) = hanni(j, 0)
                    nyuuryokuhanni(g - 1, 1, k - 1) = hanni(j, 1)
                    Exit For
                Else
                    nyuuryokuhanni(g - 1, 0, k - 1) = 1000
                    nyuuryokuhanni(g - 1, 1, k - 1) = 1000
                End If
            Next j
        Next g
    Next k
    
    sidou = Empty
    siido = Empty
    s = 0
    
    '全テーブル検証
    For b = 0 To 16 Step 1
        
        'ここからテーブルデータと照合
        furagu = 0
        sido = Empty
        
        '全シード値検証
        For gyou = 1 To maxl(b) Step 1
            
            'シード値が空なら抜ける
            If IsEmpty(seed(gyou, b + 1)) Then
                Exit For
            End If
            
            ok = 0
            kai = 0
            
            '全回検証
            For k = 1 To pata Step 1
                
                'スキル判別
                If skill(k, 1) = "激運" Then
                    lack = 29
                ElseIf skill(k, 1) = "幸運" Then
                    lack = 26
                Else
                    lack = 22
                End If
                
                mawari = 0
                
                '報酬判定
                For n = 0 To waku - 1 Step 1
                    
                    mawari = mawari + 1
                    
                    If n > 2 And seed(((gyou + kai + n * 2 - 1) Mod maxl(b)) + 1, b + 1) Mod 32 >= lack Then
                        If nyuuryokuhanni(n, 1, k - 1) = 1000 Then
                            ok = 1
                        Else
                            ok = 0
                        End If
                        Exit For
                    End If
                    
                    mawari = mawari + 1
                    
                    wari = (gyou + kai + n * 2) Mod maxl(b)
                    x = seed(wari + 1, b + 1) Mod 100
                    
                    If (nyuuryokuhanni(n, 0, k - 1) <= x And nyuuryokuhanni(n, 1, k - 1) >= x) Then
                        ok = 1
                    Else
                        ok = 0
                        Exit For
                    End If
                    
                Next n
                
                If ok = 0 Then Exit For
                
                kai = kai + mawari + kotei
                
            Next k
            
            If ok <> 0 Then
                furagu = furagu + 1
                sido = sido & seed(gyou, b + 1) & " "
                Exit For
            End If
            
        Next gyou
        
        If furagu <> 0 Then
            kekka(b) = "○"
            sidou = sidou & "T" & b + 1 & " "
            siido = siido & sido & " "
            s = s + 1
        Else
            kekka(b) = "×"
        End If
        
    Next b
    
    If s > 1 Then
        siido = "-"
    ElseIf s = 0 Then
        siido = "-"
        sidou = "該当なし"
    End If
    
    Worksheets("Sheet1").Cells(19, 7).Value = sidou
    Worksheets("Sheet1").Cells(18, 7).Value = siido
    
    Worksheets("Sheet1").Range(Worksheets("Sheet1").Cells(3, 14), Worksheets("Sheet1").Cells(3, 30)).Value = kekka
    
    Application.Calculation = xlAutomatic
    
End Sub
```
