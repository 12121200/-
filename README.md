Private Sub CommandButton1_Click()
    ' шифровка
    m = TextBox1.Value
    n = Len(m)
    c = ""
    k = TextBox4.Value
    
    If TextBox1.Text = "" Then MsgBox "Введите текст": GoTo 1
    If TextBox4.Text = "" Then MsgBox "Введите ключ": GoTo 1
    
    For i = 1 To n
        x = Ca(Mid(m, i, 1), k)
        c = c & x
    Next i
    
    TextBox2.Value = c
    
1:
End Sub

Private Function Ca(A, Sh)
    Dim ch As Integer
    ch = Asc(A)
    
    Select Case ch
        Case 224 To 255     ' строчные буквы
            Ca = Chr((ch - 224 + Sh) Mod 32 + 224)
        Case 192 To 223     ' прописные буквы
            Ca = Chr((ch - 192 + Sh) Mod 32 + 192)
        Case Else
            Ca = A
    End Select
End Function

Private Function ModMew(A, B)
    If A >= 0 Then
        ModMew = A Mod B
    Else
        ModMew = (A Mod B) + B
    End If
End Function

Private Sub CommandButton2_Click()
    ' расшифровка
    c = TextBox2.Value
    m = ""
    k = TextBox4.Value
    
    If TextBox2.Text = "" Then MsgBox "Введите шифротекст": GoTo 1
    If TextBox4.Text = "" Then MsgBox "Введите ключ": GoTo 1
    
    n = Len(c)
    
    For i = 1 To n
        x = Ca(Mid(c, i, 1), -k)   ' обратный сдвиг
        m = m & x
    Next i
    
    TextBox3.Value = m
    
1:
End Sub
