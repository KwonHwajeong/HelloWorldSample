### Delphi, 실행 파일에 리소스(폰트 파일 등) 포함하기

#### 리소스 파일 추가 하기
델파이에서 프로젝트 만든 후 메뉴에서 Project → Resources and Images → Add(폰트 파일추가)1
* Resource_type : RCDATA , Resource_identifier : Resource_1(고유한 이름)

#### 리소스 파일 사용 예제
```
procedure TForm1.FormCreate(Sender: TObject);
begin
  _LoadFont;
end;

procedure TForm1.FormDestroy(Sender: TObject);
begin
  _RemoveFont;
end;

procedure TForm1._LoadFont;
var
  ResStream: TResourceStream;
  sFileName: string;
begin
  sFileName := ExtractFilePath(Application.ExeName) + 'D2Coding.ttc';

  ResStream := nil;
  try
    ResStream := TResourceStream.Create(hInstance, 'Resource_1', RT_RCDATA);
    try
      ResStream.SaveToFile(sFileName);
    except
      on E: EFCreateError do // on E : Exception do
        ShowMessage(E.Message);
    end;
  finally
    ResStream.Free;
  end;

  AddFontResource(PChar(sFileName));
  SendMessage(HWND_BROADCAST, WM_FONTCHANGE, 0, 0);
end;

procedure TForm1._RemoveFont;
var
  sFile: string;
begin
  sFile := ExtractFilePath(Application.ExeName) + 'D2Coding.ttc';
  if FileExists(sFile) then
  begin
    RemoveFontResource(PChar(sFile));
    SendMessage(HWND_BROADCAST, WM_FONTCHANGE, 0, 0);
    DeleteFile(sFile);
  end;
end;
```

https://stackoverflow.com/questions/556147/how-to-quickly-and-easily-embed-fonts-in-winforms-app-in-c-sharp

https://stackoverflow.com/questions/15949057/addfontfile-from-resources
