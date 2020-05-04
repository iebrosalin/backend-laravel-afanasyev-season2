# Шпаргалка на основе курса


## Установка через composer

````
composer create-project --prefer-dist laravel/laravel=версия папка
````

## Плагины для PhpStorm
- Laravel Plugin

## Пакеты без которых жизнь не мила:
```
composer require barryvdh/laravel-debugbar --dev
composer require --dev barryvdh/laravel-ide-helper
```


## Создание моделей 

```
php artisan make:model Model/BlogCategory -m

php artisan make:model Model/BlogPost -m<
```
## Создание БД для Laravel

````
create schema `poligon` default character set utf8mb4 collate utf8mb4_unicode_ci;
````

## Миграция 

````
php artisan migrate
````

## Создание сидеров, фабрик

```
php artisan make:seeder UserTableSeeder

php artisan make:seeder BlogCategoriesTableSeeder
```

Запуск сидов

```

php artisan db:seed
php artisan db:seed --class=UserTableSeeder
php artisan migrate:refresh --seed

```

Содание фабрики

```

php artisan make:factory BlogPostFactory --model="App\Models\BlogPost"

```

При создании новых сидов полезно делать 

```

composer dumpautoload

```

## Контроллеры

Создание

```

php artisan make:controller RestTestController --resource

```

Аутентификация (для базовой вёрстки)
```
php artisan make:auth
php artisan migrate
```

## Примеры выборки
```
        $item = BlogCategory::findOrFail($id);
//        $item = BlogCategory::find($id);
//        $item []= BlogCategory::findOrFail($id);
//        $item []= BlogCategory::where('id','=' ,$id)->first();
//        $item []= BlogCategory::where('id','=' ,$id)->get(); // возвращает array
//        $item []= BlogCategory::where('id', $id)->first();
//        dd($item);
```

## Создание своего Request

```
php artisan make:request BlogCategoryUpdateRequest
```

## Говнокод для валидации в контроллере

```
    use Illuminate\Foundation\Validation\ValidatesRequests;

//        $rules = [
//          'title' => 'required|min:5|max:200',
//          'slug' => 'max:191',
//          'description' => 'string|min:3|max:500',
//          'parent_id' => 'required|integer|exists:blog_categories,id',
//        ];

//        $validatedData = $this->validate($request, $rules);

//        $validatedData = $request->validate($rules);

     /*   $validator = \Validator::make($request->all(), $rules);

        $validatedData [] = $validator->passes();
        $validatedData [] = $validator->validate();
        $validatedData [] = $validator->valid();
        $validatedData [] = $validator->failed();
        $validatedData [] = $validator->errors();
        $validatedData [] = $validator->fails();

        dd($validatedData);

        */
```


## Сохранение объекта

```
//        $item = BlogCategory($data);
//        $item->save();

        $item = (new BlogCategory())->create($data);
```

## Примеры запросов

```
        $columns = implode(', ', [
            'id',
            'CONCAT (id, ". ", title) AS id_title'
        ]);

//        $result [] = $this->startConditions()->all();
//        $result [] = $this
//            ->startConditions()
//            ->select('blog_categories.*',
//                DB::raw('CONCAT (id, ". ", title) AS id_title'))
//            ->toBase()
//            ->get();

        $result [] = $this
            ->startConditions()
            ->selectRaw($columns)
            ->toBase()
            ->get();

```


## Отношения

Объявление отношения в классе

```
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class BlogPost extends Model
{
    use SoftDeletes;

    /**
     * @return \Illuminate\Database\Eloquent\Relations\BelongsTo
     */
    public function category()
    {
        return $this->belongsTo(BlogCategory::class);
    }

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    protected $dates = [
        'published_at'
    ];
}

```

Жадная загрузка

```
    public function getAllWithPaginate($perPage = null)
    {
        $columns = [
            'id',
            'title',
            'slug',
            'is_published',
            'published_at',
            'user_id',
            'category_id'
        ];

        $result = $this
            ->startConditions()
            ->select($columns)
            ->orderBy('id', 'DESC')
//            ->with(['category', 'user'])
            ->with(
                [
                    'category' => function($query) {
                        $query->select(['id', 'title']);
                    },
                    'user:id,name'
                ]
            )
            ->paginate($perPage);

        return $result;
    }
```

Ленивая загрузка

```
 <td>{{ $post->user->name }}</td>
 <td>{{ $post->category->title }}</td>
```

## Observers

Создание

```
php artisan make:observer BlogPostObserver --model=Models\BlogPost
php artisan make:observer BlogCategoryObserver --model=Models\BlogCategory
```

Полезные методы для работы в обсервере

```
        $test[] = $blogPost->isDirty();
//        $test[] = $blogPost->isDirty('is_published');
//        $test[] = $blogPost->isDirty('user_id');
//        $test[] = $blogPost->getAttribute('is_published');
//        $test[] = $blogPost->is_published;
//        $test[] = $blogPost->getOriginal('is_published');
//        dd($test)
```

## Методы выводы информации о поле объекта

```
        $v ['title_before'] = $item->title;

        $item->title = '';

        $v['title_after'] = $item->title;
        $v['getAttribute'] = $item->getAttributes('title');
        $v['attributesToArray'] = $item->getAttributesToArray('title');
        $v['attributes'] = $item->attributes['title'];
        $v['getAttributeValue'] = $item->getAttributeValue('title');
        $v['getMutatedAttributes'] = $item->getMutatedAttributes();
        $v['hasGetMutator for title'] = $item->hasGetMutator('title');
        $v['toArray'] = $item->toArray();

        dd($v, $item);
```

## Немного мутатор и аксессоров

```
//    public function getTitleAttribute($valueFromObject)
//    {
//        return mb_strtoupper($valueFromObject);
//    }
//
//    public function setTitleAttribute($incomingValue)
//    {
//        $this->attributes['title'] = mb_strtolower($incomingValue);
//    }
```

## Удаление 

```
        $result = BlogPost::destroy($id);

//        $result = BlogPost::find($id)->forceDelete();
```

## Очереди

Комманды создания таблицы для очередей

- до 6.0

```
php artisan queue:table
php artisan meigrate
```


- в 6.0

```
php artisan queue:table
php artisan queue:failed-table
php artisan meigrate
```

Создание job

```
php artisan make:job NameClass
```

Запуск очереди

```
/*
    Запускает процесс обработки задач очереди как демон. Все изменения сделанные в коде приняты не 
    будут, потреьбуется перезапуск вручную.
*/
php artisan queue:work

/*
    Сначала выпоняются все задачи из очереди queueName1, а потом queueName2
*/
php artisan queue:work --queue=queueName1,queueName2

/*
    Запускает процесс обработки задач очереди. Применяет изменения в коде. Хуже производительность 
    по сравнению queue:work. 
*/
php artisan queue:listen

/*
    Мягкий перезапуск демона queue:work после того как завершит последнюю задачу
*/
php artisan queue:restart


/*
    Просмотр таблицы проваленных задач
*/
php artisan queue:failed

/*
    Возврат в очередь выполнения всех проваленных задач
*/
php artisan queue:retry all

/*
    Возврат в очередь выполнения проваленной задачи с id 5
*/
php artisan queue:retry 5
```
