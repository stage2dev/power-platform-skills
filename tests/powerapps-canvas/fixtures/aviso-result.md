Screens:
  Tela Exemplo:
    Properties:
      Fill: =fxColorBackground
      Height: =1080
      Width: =1920
    Children:
      - ex-lbl-x:
          Control: Label@2.5.1
          Properties:
            Text: =First(Split("a;b", ";")).Result
