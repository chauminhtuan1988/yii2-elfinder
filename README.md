ElFinder pаckage for Yii 2
===========================

ElFinder — file manager for website.

## Поддерживаемые хранилища

mihaildev/yii2-elfinder-flysystem - https://github.com/MihailDev/yii2-elfinder-flysystem/

```
    Local
    Azure
    AWS S3 V2
    AWS S3 V3
    Copy.com
    Dropbox
    FTP
    GridFS
    Memory
    Null / Test
    Rackspace
    ReplicateAdapter
    SFTP
    WebDAV
    PHPCR
    ZipArchive
```


## Installation

The most convenient way to install this extension is through [composer](http://getcomposer.org/download/).

Or run

```
php composer.phar require --prefer-dist mihaildev/yii2-elfinder "*"
```

or add

```json
"mihaildev/yii2-elfinder": "*"
```

в разделе `require` вашего composer.json файла.

## Setting

```php
'controllerMap' => [
        'elfinder' => [
            'class' => 'mihaildev\elfinder\Controller',
            'access' => ['@'], //global access to file manager @ - for authorized, ? - for guests, to open to everyone ['@', '?']
            'disabledCommands' => ['netmount'], //disabling unnecessary commands https://github.com/Studio-42/elFinder/wiki/Client-configuration-options#commands
            'roots' => [
                [
                    'baseUrl'=>'@web',
                    'basePath'=>'@webroot',
                    'path' => 'files/global',
                    'name' => 'Global'
                ],
                [
                    'class' => 'mihaildev\elfinder\volume\UserPath',
                    'path'  => 'files/user_{id}',
                    'name'  => 'My Documents'
                ],
                [
                    'path' => 'files/some',
                    'name' => ['category' => 'my','message' => 'Some Name'] //перевод Yii::t($category, $message)
                ],
                [
                    'path'   => 'files/some',
                    'name'   => ['category' => 'my','message' => 'Some Name'], // Yii::t($category, $message)
                    'access' => ['read' => '*', 'write' => 'UserFilesAccess'] // * - for everyone, otherwise the access check in this example everyone can see and only users with rights can edit UserFilesAccess
                ]
            ],
            'watermark' => [
            		'source'         => __DIR__.'/logo.png', // Path to Water mark image
                     'marginRight'    => 5,          // Margin right pixel
                     'marginBottom'   => 5,          // Margin bottom pixel
                     'quality'        => 95,         // JPEG image save quality
                     'transparency'   => 70,         // Water mark image transparency ( other than PNG )
                     'targetType'     => IMG_GIF|IMG_JPG|IMG_PNG|IMG_WBMP, // Target image formats ( bit-field )
                     'targetMinPixel' => 200         // Target image minimum pixel size
            ]
        ]
    ],
```

```php
'controllerMap' => [
        'elfinder' => [
			'class' => 'mihaildev\elfinder\PathController',
			'access' => ['@'],
			'root' => [
				'path' => 'files',
				'name' => 'Files'
			],
			'watermark' => [
						'source'         => __DIR__.'/logo.png', // Path to Water mark image
						 'marginRight'    => 5,          // Margin right pixel
						 'marginBottom'   => 5,          // Margin bottom pixel
						 'quality'        => 95,         // JPEG image save quality
						 'transparency'   => 70,         // Water mark image transparency ( other than PNG )
						 'targetType'     => IMG_GIF|IMG_JPG|IMG_PNG|IMG_WBMP, // Target image formats ( bit-field )
						 'targetMinPixel' => 200         // Target image minimum pixel size
			]
		]
    ],
```

The difference between PathController and Controller is that PathController works only with one folder and also has the additional ability to pass in the request to open subdirectories


At the moment, only Local FileSystem storage is used. (mihaildev\elfinder\volume\Local And mihaildev\elfinder\volume\UserPath)
to use the rest you will have to configure everything through mihaildev\elfinder\volume\Base
also added extension  https://github.com/MihailDev/yii2-elfinder-flysystem/ This add-on allows you to integrate Flysystem repositories such as
    Local
    Azure
    AWS S3 V2
    AWS S3 V3
    Copy.com
    Dropbox
    FTP
    GridFS
    Memory
    Null / Test
    Rackspace
    ReplicateAdapter
    SFTP
    WebDAV
    PHPCR
    ZipArchive

## Setting Up Plugins
Due to the complex setup, the plugins were redesigned, but the ability to use old plugins is present
```php
'controllerMap' => [
        'elfinder' => [
            'class' => 'mihaildev\elfinder\Controller',
            //'plugin' => ['\mihaildev\elfinder\plugin\Sluggable'],
            'plugin' => [
                [
                    'class'=>'\mihaildev\elfinder\plugin\Sluggable',
                    'lowercase' => true,
                    'replacement' => '-'
                ]
             ],
             'roots' => [
                             [
                                 'baseUrl'=>'@web',
                                 'basePath'=>'@webroot',
                                 'path' => 'files/global',
                                 'name' => 'Global',
                                 'plugin' => [
                                        'Sluggable' => [
                                            'lowercase' => false,
                                        ]
                                 ]
                             ],
                         ]

```

Setting up an old plugin (using the plugin as an example Sanitizer)
```php
'controllerMap' => [
        'elfinder' => [
            'class' => 'mihaildev\elfinder\Controller',
            'connectOptions' => [
                'bind' => [
                    'upload.pre mkdir.pre mkfile.pre rename.pre archive.pre ls.pre' => array(
                        'Plugin.Sanitizer.cmdPreprocess'
                    ),
                    'ls' => array(
                        'Plugin.Sanitizer.cmdPostprocess'
                    ),
                    'upload.presave' => array(
                        'Plugin.Sanitizer.onUpLoadPreSave'
                    )
                ],
                'plugin' => [
                    'Sanitizer' => array(
                        'enable' => true,
                        'targets'  => array('\\','/',':','*','?','"','<','>','|'), // target chars
                        'replace'  => '_'    // replace to this
                    )
                ],
            ],


             'roots' => [
                             [
                                 'baseUrl'=>'@web',
                                 'basePath'=>'@webroot',
                                 'path' => 'files/global',
                                 'name' => 'Global',
                                 'plugin' => [
                                        'Sanitizer' => array(
                                                                'enable' => true,
                                                                'targets'  => array('\\','/',':','*','?','"','<','>','|'), // target chars
                                                                'replace'  => '_'    // replace to this
                                                            )
                                 ]
                             ],
                         ]

```

## Usage

```php
use mihaildev\elfinder\InputFile;
use mihaildev\elfinder\ElFinder;
use yii\web\JsExpression;

echo InputFile::widget([
    'language'   => 'vi',
    'controller' => 'elfinder', // insert the name of the controller, by default it is elfinder
    'filter'     => 'image',    // file filter, you can specify an array of filters https://github.com/Studio-42/elFinder/wiki/Client-configuration-options#wiki-onlyMimes
    'name'       => 'myinput',
    'value'      => '',
]);

echo $form->field($model, 'attribute')->widget(InputFile::className(), [
    'language'      => 'vi',
    'controller'    => 'elfinder', // insert the name of the controller, by default it is elfinder
    'filter'        => 'image',    // file filter, you can specify an array of filters https://github.com/Studio-42/elFinder/wiki/Client-configuration-options#wiki-onlyMimes
    'template'      => '<div class="input-group">{input}<span class="input-group-btn">{button}</span></div>',
    'options'       => ['class' => 'form-control'],
    'buttonOptions' => ['class' => 'btn btn-default'],
    'multiple'      => false       // ability to select multiple files
]);

echo ElFinder::widget([
    'language'         => 'vi',
    'controller'       => 'elfinder', // insert the name of the controller, by default it is elfinder
    'filter'           => 'image',    // file filter, you can specify an array of filters https://github.com/Studio-42/elFinder/wiki/Client-configuration-options#wiki-onlyMimes
    'callbackFunction' => new JsExpression('function(file, id){}') // id - id widget
]);

```

## Use when working with PathController
```php
use mihaildev\elfinder\InputFile;
use mihaildev\elfinder\ElFinder;
use yii\web\JsExpression;

echo InputFile::widget([
    'language'   => 'vi',
    'controller' => 'elfinder', // insert the name of the controller, by default it is elfinder
    'path' => 'image', // the folder from the controller settings will be opened with the specified subdirectory added
    'filter'     => 'image',    // file filter, you can specify an array of filters https://github.com/Studio-42/elFinder/wiki/Client-configuration-options#wiki-onlyMimes
    'name'       => 'myinput',
    'value'      => '',
]);

echo $form->field($model, 'attribute')->widget(InputFile::className(), [
    'language'      => 'vi',
    'controller'    => 'elfinder', // insert the name of the controller, by default it is elfinder
    'path' => 'image', // будет открыта папка из настроек контроллера с добавлением указанной под деритории 
    'filter'        => 'image',    // фильтр файлов, можно задать массив фильтров https://github.com/Studio-42/elFinder/wiki/Client-configuration-options#wiki-onlyMimes
    'template'      => '<div class="input-group">{input}<span class="input-group-btn">{button}</span></div>',
    'options'       => ['class' => 'form-control'],
    'buttonOptions' => ['class' => 'btn btn-default'],
    'multiple'      => false       // возможность выбора нескольких файлов
]);

echo ElFinder::widget([
    'language'         => 'vi',
    'controller'       => 'elfinder', // insert the name of the controller, by default it is elfinder
    'path' => 'image', // the folder from the controller settings will be opened with the specified subdirectory added
    'filter'           => 'image',    // file filter, you can specify an array of filters https://github.com/Studio-42/elFinder/wiki/Client-configuration-options#wiki-onlyMimes
    'callbackFunction' => new JsExpression('function(file, id){}') // id - id widget
]);

```

## CKEditor
```php
use mihaildev\elfinder\ElFinder;

$ckeditorOptions = ElFinder::ckeditorOptions($controller,[/* Some CKEditor Options */]);

```

To specify a subdirectory (when using PathController)
```php
use mihaildev\elfinder\ElFinder;

$ckeditorOptions = ElFinder::ckeditorOptions([$controller, 'path' => 'some/sub/path'],[/* Some CKEditor Options */]);

```

Use in conjunction with the application "mihaildev/yii2-ckeditor" (https://github.com/MihailDev/yii2-ckeditor)

```php
use mihaildev\ckeditor\CKEditor;
use mihaildev\elfinder\ElFinder;

$form->field($model, 'attribute')->widget(CKEditor::className(), [
  ...
  'editorOptions' => ElFinder::ckeditorOptions('elfinder',[/* Some CKEditor Options */]),
  ...
]);
```

To specify a subdirectory (when using PathController)

```php
use mihaildev\ckeditor\CKEditor;
use mihaildev\elfinder\ElFinder;

$form->field($model, 'attribute')->widget(CKEditor::className(), [
  ...
  'editorOptions' => ElFinder::ckeditorOptions(['elfinder', 'path' => 'some/sub/path'],[/* Some CKEditor Options */]),
  ...
]);
```

## Problems
When embedding without iframe, there may be a conflict with bootstrap.js. Studio-42/elFinder#740
Solution - add a record to the template
```php

mihaildev\elfinder\Assets::noConflict($this);

```

## Useful links

ElFinder Wiki - https://github.com/Studio-42/elFinder/wiki

Flysystem

https://github.com/MihailDev/yii2-elfinder-flysystem/

https://github.com/barryvdh/elfinder-flysystem-driver

https://github.com/creocoder/yii2-flysystem

http://flysystem.thephpleague.com/




