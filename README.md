> 需求:
  排程自動將檔案上傳至google drive試算表
  
## 解決方案
至GCP新增專案並啟用drive api，並新增service account憑證  
注意Oauth用戶端ID憑證也可以上傳，網路上這部分的資訊應該比較多，但會有使用者必須按下同意的畫；  
此案例是server to server排程自動完成，不可能每次都讓使用者去按，所以必須選擇service account方案
service account可以建立json檔案認證

1. 至GCP開啟或新增專案  

2. 啟用Drive API 
![2-啟用API和服務](https://user-images.githubusercontent.com/24542187/216818699-3c767680-c1ae-4186-9e28-7c7699d51744.png)
![2 2選取google-drive-api](https://user-images.githubusercontent.com/24542187/216818866-3d042d22-353c-4502-aebf-e3392d0cd5de.png)
![2 3-啟用google-drive-api服務](https://user-images.githubusercontent.com/24542187/216818949-7bcb4c5a-b5cd-4eff-97ab-4d7081184fb2.png)

3. 建立service account
![3 1-建立service-account](https://user-images.githubusercontent.com/24542187/216819018-234a16e6-b847-441b-a222-07f95998869f.png)
![3 2-建立服務帳戶](https://user-images.githubusercontent.com/24542187/216819029-33c30135-d273-419d-a494-3ed172907ea2.png)
![3 3-建立服務帳號](https://user-images.githubusercontent.com/24542187/216819033-82799b92-dae2-42e2-aab0-5d21c00f3ffe.png)

4. 建立服務帳號金鑰  
![4 1-在服務帳號裡建立新的金鑰](https://user-images.githubusercontent.com/24542187/216819622-62a5f9b1-cdb2-495d-ae41-1f01a514782d.png)
![4 2-建立金鑰選json](https://user-images.githubusercontent.com/24542187/216819623-d8e3f5f2-6b70-4228-a0a2-7069e2b1657e.png)

5. 分享檔案權限給該service account
![5-drive-share-file](https://user-images.githubusercontent.com/24542187/216819982-461f2f5a-6307-4453-ad14-1f15861ce77b.png)

## 安裝 php client library for google api
```
composer require google/apiclient:^2.12.1
```

library 網址: https://github.com/googleapis/google-api-php-client

## sample code
1. list file 範例
```
<?php
require_once __DIR__ . '/vendor/autoload.php';

putenv('GOOGLE_APPLICATION_CREDENTIALS=drive-api-374810-57dd2341234.json');

use Google\Client;
use Google\Service\Drive;


$client = new Client();
$client->useApplicationDefaultCredentials();
//$client->addScope(Google_Service_Drive::DRIVE);
$client->setScopes(array('https://www.googleapis.com/auth/drive'));

$service = new Drive($client);

$folderId = '1jtgSWYiiSs2PO2YDtHOk7AZAlzq_7abc';

$optParams = array(
    'pageSize' => 10,
    'fields' => "nextPageToken, files(contentHints/thumbnail,fileExtension,iconLink,id,name,size,thumbnailLink,webContentLink,webViewLink,mimeType,parents)",
    'q' => "'" . $folderId . "' in parents"
);
$results = $service->files->listFiles($optParams);

// header('Content-Type: application/json'); //to beautify view in browser
// echo json_encode($response);
var_dump($results);
```

2. 上傳檔案
```
<?php
require_once __DIR__ . '/vendor/autoload.php';

putenv('GOOGLE_APPLICATION_CREDENTIALS=drive-api-374810-57dd81231234.json');

use Google\Client;
use Google\Service\Drive;

$client = new Client();
$client->useApplicationDefaultCredentials();
// $client->setScopes('https://www.googleapis.com/auth/drive');

$client->addScope(Drive::DRIVE);

$driveService = new Drive($client);

$fileMetadata = new Drive\DriveFile(array(
    'name' => 'dog.jpeg',
    'parents' => ['1jtgSWYiiSs9PO2YDtHOk7AZAlzq_7abc'],
    'mimeType' => 'image/jpeg',
));


$content = file_get_contents('dog.jpeg');

$file = $driveService->files->create($fileMetadata, array(
    'data' => $content,
    'mimeType' => 'image/jpeg',
    'uploadType' => 'multipart'
));
var_dump($file);
```

3. 修改檔案
如果上個範例的upload同樣檔名，會出現很多個一模一樣的檔案，要用修改才能覆蓋本來的檔案.  
此範例順便將csv轉成google sheet試算表
```
<?php
require_once __DIR__ . '/vendor/autoload.php';

putenv('GOOGLE_APPLICATION_CREDENTIALS=drive-api-374810-57dd81231234.json');

use Google\Client;
use Google\Service\Drive;

$client = new Client();
$client->useApplicationDefaultCredentials();
// $client->setScopes('https://www.googleapis.com/auth/drive');

$client->addScope(Drive::DRIVE);

$driveService = new Drive($client);

$fileMetadata = new Drive\DriveFile(array(
    'name' => 'tttt',
    // 'parents' => ['1jtgSWYiiSs0PO2YDtHOk7AZAlzq_7abc'],
    'mimeType' => 'application/vnd.google-apps.spreadsheet',
));


$content = file_get_contents('20221028150625.csv');

$fileId = '123BS3z_xPb4ZbyykF4dLuAFUHEXfypf1efqJabckefg';
$file = $driveService->files->update($fileId, $fileMetadata, array(
    'data' => $content,
    'mimeType' => 'text/csv',
    'uploadType' => 'multipart',
    'fields' => 'id'
));
var_dump($file);
exit;
printf("File ID: %s\n", $file->id);
```

官方網站:  
https://developers.google.com/drive/api/guides/manage-uploads?hl=zh-tw  
