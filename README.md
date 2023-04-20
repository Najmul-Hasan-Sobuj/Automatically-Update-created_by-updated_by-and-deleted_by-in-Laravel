# Automatically Update created_by updated_by and deleted_by in Laravel

1. Create the migration file for the posts table by running the following command in your terminal:
```
php artisan make:migration create_posts_table --create=posts
```

2. In the generated migration file, replace the contents with the following code:
```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreatePostsTable extends Migration
{
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->text('body');
            $table->unsignedBigInteger('created_by')->nullable();
            $table->unsignedBigInteger('updated_by')->nullable();
            $table->unsignedBigInteger('deleted_by')->nullable();
            $table->softDeletes();
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('posts');
    }
}
```

3. Run the migration to create the `posts` table:
```
php artisan migrate
```

4. Create the `Post` model by running the following command in your terminal:
```
php artisan make:model Post
```

5. In the generated model file `app/Models/Post.php`, add the following code:
```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Schema;

class Post extends Model
{
    use HasFactory, SoftDeletes;

    protected static function boot()
    {
        parent::boot();

        static::saving(function ($model) {
            if (Auth::check()) {
                $authId = Auth::id();
                $hasCreated = Schema::hasColumn($model->getTable(), 'created_by');
                $hasUpdated = Schema::hasColumn($model->getTable(), 'updated_by');
                $hasDeleted = Schema::hasColumn($model->getTable(), 'deleted_by');

                if ($hasCreated && !$model->exists) {
                    $model->created_by = $authId;
                }

                if ($hasUpdated && $model->exists) {
                    $model->updated_by = $authId;
                }

                if ($hasDeleted && $model->isForceDeleting()) {
                    $model->deleted_by = $authId;
                }
            }
        });
    }
}
```

6. Create the `PostFactory` by running the following command in your terminal:
```
php artisan make:factory PostFactory --model=Post
```

7. In the generated factory file `database/factories/PostFactory.php`, replace the contents with the following code:
```php
namespace Database\Factories;

use App\Models\Post;
use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

class PostFactory extends Factory
{
    protected $model = Post::class;

    public function definition()
    {
        return [
            'title' => $this->faker->sentence,
            'body'  => $this->faker->paragraphs(3, true),
            'created_by'    =>  Auth::id() ?? 1, 
            'updated_by'    =>  Auth::id() ?? 1, 
            'deleted_by'    =>  null, 
        ];
    }
}
```
