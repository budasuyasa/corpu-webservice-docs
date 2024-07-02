# Dokumentasi Integrasi Elearning Corporate University

WIKI ini menjelaskan tentang integrasi Elearning Corporate University dengan aplikasi pihak ketiga.

Elearning Corporate University dikembangkan dengan menggunakan Moodle versi 4.3. Untuk melakukan integrasi dengan aplikasi pihak ketiga, Elearning Corporate University menyediakan REST API yang dapat digunakan untuk melakukan operasi-operasi yang berkaitan dengan operasi umum seperti membuat kategori course, membuat course, menambahkan partisipan dan mendapatkan nilai hasil belajar.

## Prasayarat

WIKI ini ditulis untuk pengembang yang menggunakan bahasa pemrograman PHP. Sehingga, pengembang diharapkan memiliki pengetahuan dasar tentang PHP serta dapat mempersiapkan beberapa hal berikut:

1. Access Token yang dapat dibuat melalui menu `Site administration > Plugins > Web services > Manage tokens`.
2. URL Elearning Corporate University yang akan diakses.
3. composer untuk menginstall library Guzzle dan PHPDotEnv.

## Instalasi

Untuk melakukan instalasi, pengembang perlu menginstall library Guzzle. Library ini digunakan untuk melakukan request ke REST API Elearning Corporate University. Berikut adalah langkah-langkah instalasi:

1. Buka terminal dan arahkan ke direktori project pengembang.
2. Jalankan perintah berikut untuk menginstall library Guzzle dan PHPDotEnv:

```bash
composer require guzzlehttp/guzzle
composer require vlucas/phpdotenv
```

3. Buat file `.env` pada direktori project dan isi dengan konfigurasi berikut:

```env
LMS_URL=https://balcorpu.baliprov.go.id/webservice/rest/server.php
LMS_TOKEN=isi_dengan_access_token
```


## Membuat Kategori

Untuk dapat membuat course category, kita bisa menggunakan snippet berikut:

```php
use GuzzleHttp\Client;

$client = new Client([
    'base_uri' => env('LMS_URL'),
]);

$categoryName = 'Category Name';
$categoryIdNumber = '0001';

$params = [
    'wstoken' => env('LMS_TOKEN'),
    'wsfunction' => 'core_course_create_categories',
    'moodlewsrestformat' => 'json',
    'categories' => [
        [
            'name' => $categoryName,
            'idnumber' => $categoryIdNumber,
        ]
    ]
];

$response = $client->post('', ['form_params' => $params]);
$data = json_decode($response->getBody(), true);

if (isset($data['exception'])) {
    return response()->json(['message' => 'Error: ' . $data['message']], 500);
}

return response()->json(['message' => 'Category created successfully.'], 200);
```

## Membuat Course

Untuk membuat course, kita bisa menggunakan snippet berikut:

```php

use GuzzleHttp\Client;

$client = new Client([
    'base_uri' => env('LMS_URL'),
]);
$categoryid = '0001';

$params = [
    'wstoken' => env('LMS_TOKEN'),
    'wsfunction' => 'core_course_create_courses',
    'moodlewsrestformat' => 'json',
    'courses' => [
        [
            // Seseuaikan value ini sesuai dengan detail course yang dibuat
            'shortname' => 'Course Shortname',
            'fullname' => 'Course Fullname',
            'idnumber' => 'Course ID Number',
            'categoryid' => $categoryid,
        ]
        ]
];

$response = $client->post('', ['form_params' => $params]);
$data = json_decode($response->getBody(), true);

if (isset($data['exception'])) {
    return response()->json(['message' => 'Error: ' . $data['message']], 500);
}

return response()->json(['message' => 'Course created successfully.'], 200);
```

### Menambahkan User Baru ke Elearning

```php

use GuzzleHttp\Client;

$client = new Client([
    'base_uri' => env('LMS_URL'),
]);


$params = [
    'wstoken' => env('LMS_TOKEN'),
    'wsfunction' => 'core_user_create_users',
    'moodlewsrestformat' => 'json',
    'users' => [
        [
            'username' => 'username',
            'password' => 'password',  
            'firstname' => 'Firstname',
            'lastname' => 'Lastname',
            'email' => 'pengguna@domain.com',
            'auth' => 'manual',
            'idnumber' => '',
            'lang' => 'id',
            'timezone' => 'Asia/Makassar',
            'mailformat' => 0,
            'description' => '',
            'city' => 'Denpasar',
            'country' => 'ID',  
        ]
    ]
];

$response = $client->post('', ['form_params' => $params]);
$data = json_decode($response->getBody(), true);

if (isset($data['exception'])) {
    return response()->json(['message' => 'Error: ' . $data['message']], 500);
}

return response()->json(['message' => 'User created successfully.'], 200);
```


### Menambahkan User sebagai Partisipan

```php
$client = new Client([
    'base_uri' => env('LMS_URL'),
]);

// Mendapatkan userid berdasarkan username
$userParams = [
    'wstoken' => env('LMS_TOKEN'),
    'wsfunction' => 'core_user_get_users_by_field',
    'moodlewsrestformat' => 'json',
    'field' => 'username',
    'values' => ['username_user'], // sesuaikan dengan value username
];

$userResponse = $client->post('', ['form_params' => $userParams]);
$userData = json_decode($userResponse->getBody(), true);

if (empty($userData)) {
    return response()->json(['message' => 'User not found.'], 404);
}

$userId = $userData[0]['id'];

$params = [
    'wstoken' => env('LMS_TOKEN'),
    'wsfunction' => 'core_course_get_courses_by_field',
    'moodlewsrestformat' => 'json',
    'field' => 'idnumber',
    'value' => 'idnumbercourse', // sesuaikan dengan idnumbercourse
];

$response = $client->post('', ['form_params' => $params]);
$data = json_decode($response->getBody(), true);

if (!empty($data['courses'])) {
    $course = $data['courses'][0];

    // enrol sesuai dengan role (3 = teacher, 5 = student)
    $roleid = $request->user_type == 'teacher' ? 3 : 5;

    $enrolmentParams = [
        'wstoken' => env('LMS_TOKEN'),
        'wsfunction' => 'enrol_manual_enrol_users',
        'moodlewsrestformat' => 'json',
        'enrolments' => [
            [
                'roleid' => $roleid,
                'userid' => $userId,
                'courseid' => $course['id'],
            ],
        ],
    ];

    $client->post('', ['form_params' => $enrolmentParams]);

    return response()->json(['message' => 'User enrolled successfully.'], 200);
```

### Mendapatkan Nilai Hasil Belajar

```php
$client = new Client();
$response = $client->request('POST', env('LMS_URL'), [
    'form_params' => [
        'wsfunction' => 'gradereport_user_get_grade_items',
        'wstoken' => env('LMS_TOKEN'),
        'moodlewsrestformat' => 'json',
        'userid' => '009', // sesuaikan dengan userid
        'courseid' => '001', // sesuaikan dengan courseid
    ]
]);
$userGrade = json_decode($response->getBody()->getContents(), true);
$res = [];
if($userGrade){
    $gradeItems = $userGrade['usergrades'][0]['gradeitems'];
    foreach($gradeItems as $gd){
        if($gd['itemname'] !== null && $gd['itemmodule'] !== 'attendance' ){
            $res[] = [
                'id' => $gd['id'],
                'nama' => $gd['itemname'],
                'nilai' => $gd['graderaw'] ? $gd['graderaw'] : 0,
                'type' => $gd['itemmodule']
            ];
        }
    }
}
return response()->json($res, 200);
```



## Catatan

Baca dokumentasi Moodle untuk mengetahui lebih lanjut tentang REST API yang disediakan oleh Moodle. Dokumentasi dapat diakses di [https://docs.moodle.org/dev/Web_service_API_functions](https://docs.moodle.org/dev/Web_service_API_functions).
Jika web service yang disediakan oleh Moodle tidak sesuai dengan kebutuhan, pengembang dapat membuat web service sendiri dengan menggunakan plugin Web Service yang disediakan oleh Moodle. Dokumentasi dapat diakses di [https://docs.moodle.org/dev/Creating_a_web_service_client](https://docs.moodle.org/dev/Creating_a_web_service_client).
Jika tidak pengembang bisa juga menggunakan Moodle Hooks untuk melakukan operasi-operasi tertentu. Dokumentasi dapat diakses di [https://docs.moodle.org/dev/Hooks](https://docs.moodle.org/dev/Hooks).
