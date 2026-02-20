Private Sub CommandButton1_Click()
    ' шифровка
    m = TextBox1.Value
    n = Len(m)
    c = ""
    k = TextBox4.Value
    
    If TextBox1.Text = "" Then
        MsgBox "Введите текст"
        Exit Sub
    End If
    
    If TextBox4.Text = "" Then
        MsgBox "Введите ключ"
        Exit Sub
    End If
    
    For i = 1 To n
        x = Ca(Mid(m, i, 1), k)
        c = c & x
    Next i
    
    TextBox2.Value = c
End Sub
