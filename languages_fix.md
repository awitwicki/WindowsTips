# WindowsTips

Иногда можно заметить что в Windows появились лишние раскладки языков

<img src="img/languages_list.png">

Но в настройках ввода их или нет, или они там не все.

<img src="img/languages_settings.png">

На разных компьютерах это провяляется по-разному, но в основном появляются раскладки типа `English (United Kingdom)`.

Самый простой способ удалить такие раскладки, которых нет в списке настроек, это добавить их вручную (тогда они появятся в списке) и после этого уже удалить их.
Важно что добавить надо именно ту раскладку которая является лишней (например именно `English (United Kingdom)`)

Далее, можно сделать себе простой powershell скрипт который это делает.
Выглядит он примерно так:
```powershell
$LangList = Get-WinUserLanguageList #getlist

$LangList[0].InputMethodTips.Add('0409:00000415') #pl programmers
$LangList[0].InputMethodTips.Add('0419:A0000419') #ru-ua
$LangList[0].InputMethodTips.Add('0409:00000409') #en-us

Set-WinUserLanguageList $LangList -Force
$LangList = Get-WinUserLanguageList

$LangList.Remove(($LangList | Where-Object LanguageTag -like 'en-US'))

$LangList = Get-WinUserLanguageList
$LangList.Remove(($LangList | Where-Object InputMethodTips -like '0409:00000409')) #en-us

Set-WinUserLanguageList $LangList -Force
```

Идентификаторы локализации (`0409:00000415`) надо вручную подобрать такие, какие вам нужны, взять их можно [здесь](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/default-input-locales-for-windows-language-packs).

1. Скрипт должен вытянуть раскладки из настроек в список

    ```powershell 
    $LangList = Get-WinUserLanguageList
    ```
    
    Так же можно просто ввести команду `Get-WinUserLanguageList` в powershell консоль и посмотреть идентификаторы раскладок
    
    *здесь будут выписаны только те которые видны в настройках переключения раскладок.

2. Скрипт должен добавить лишние раскладки в список

    ```powershell
    $LangList[0].InputMethodTips.Add('0409:00000409') #en-us
    
    $LangList[0].InputMethodTips.Add('...')
    ```

3. Применить изменения (без этого не будет работать)

    ```powershell
    Set-WinUserLanguageList $LangList -Force
    ```

4. Далее нужно опять вытянуть все раскладки в переменную, удалить лишние и применить изменения

    ```powershell
    $LangList = Get-WinUserLanguageList

    $LangList.Remove(($LangList | Where-Object LanguageTag -like 'en-US'))`

    $LangList = Get-WinUserLanguageList
    
    $LangList.Remove(($LangList | Where-Object InputMethodTips -like '0409:00000409'))
    
    Set-WinUserLanguageList $LangList -Force
    ```

    Удалять можно как через `'0409:00000409'` так и через `'en-US'`

После каждого удаления раскладки нужно вытягивать все раскладки в переенную, в консоли должно выписаться `True` что свидетельствует о успешном удалении.

Изменения должны примениться сразу после отработки скрипта.
