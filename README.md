# OneDriveFolderFiles
Функция для подключения к общей папке в облаке OneDrive Personal. Прилагаю код функции, которая по аналогии с функцией Folder.Files точно также показывает вам какие файлы имеются в папке на облаке и позволяет работать с их содержимым. Как использовать? Заходим в облако, тыкаем правой кнопкой по папке и выбираем пункт "поделиться", в появившемся меню жмем "копировать ссылку" и уже эту ссылку скармливаем моей функции. Пользуйтесь на здоровье.
OneDriveFolderFiles = (url)=>
let
     //Функция перекодирования ссылки в понятный формат для API
  fx = (t)=> Binary.ToText( Text.ToBinary( t, TextEncoding.Utf8 ), BinaryEncoding.Base64 ),
    //Здесь корневой адрес API
  API_URL = "https://api.onedrive.com/v1.0/shares/",
     //тащим путь из параметра с адресом к общей папке из облака Onedrive
  FolderUrl = url,
     //преобразовываем ссылку для получения токена для API
  UrlToBase64 = fx( FolderUrl ),
     //заменяем всякое согласно инструкции по ссылке:
     //https://docs.microsoft.com/ru-ru/onedrive/developer/rest-api/api/shares_get?view=odsp-graph-online#encoding-sharing-urls
  Replaced = Text.Replace( Text.Replace( Text.TrimEnd( UrlToBase64, "=" ), "/", "_" ), "+", "-" ),
     //формируем итоговый текстовый параметр для передачи в RelativePath
  EncodedPath = "u!" & Replaced & "/root/children",
     //тащим содержимое папки из API
  Source = Json.Document(Web.Contents( API_URL, [RelativePath = EncodedPath] ) ),
     //преобразовываем полученный JSON в табличку с содержимым папки
  TableFromRecords = Table.FromRecords( Source[value] )[[name],[webUrl]],
     //добавляем столбец с текстовыми параметрами для передачи в RelativePath
  AddColEncodedPaths = Table.AddColumn(
     TableFromRecords,
     "GetRelativePath",
     each "u!" & fx([webUrl]) & "/root/content"
  ),
     //достаем бинарники по сформированным ссылкам, дальше по аналогии как с обычными файлами с диска
  GetBinaries = Table.AddColumn(
     AddColEncodedPaths,
     "Content",
     each Web.Contents(API_URL, [RelativePath = [GetRelativePath]] ),
     Binary.Type
  )
in
  GetBinaries
