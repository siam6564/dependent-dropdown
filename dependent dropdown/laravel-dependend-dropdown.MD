# Laravel Dependent Dropdown

- Create Model with migration
`php artisan make:model Category -m`

- Custom cateory migration
```
public function up()
{
    Schema::create('categories', function (Blueprint $table) {
        $table->increments('id');
        $table->string('name');
        $table->integer('parent')->nullable();
        $table->timestamps();
    });
}

public function down()
{
    Schema::drop('categories');
}
```

- Run the migration
`php artisan migrate`

- Prepare data with seeder
`php artisan make:seeder CategoryTableSeeder`

```
// ...
class CategoryTableSeeder extends Seeder
{
    public function run()
    {
    	DB::statement('SET FOREIGN_KEY_CHECKS=0');
        DB::table('categories')->truncate();
        DB::table('categories')->insert([
        	[
        		'name' => 'Category 1',        	
        		'parent' => NULL,
        	],
        	[
        		'name' => 'Category 2',        	
        		'parent' => NULL,
        	],
        	[
        		'name' => 'Category 1.1',        	
        		'parent' => 1
        	],
        	[
        		'name' => 'Category 1.2',        	
        		'parent' => 1
        	],
        	[
        		'name' => 'Category 2.1',        	
        		'parent' => 2
        	],
        	[
        		'name' => 'Category 2.2',        	
        		'parent' => 2
        	],
        ]);
    }
}

```

- Create the route
```
 Route::get('/categories', [
  	'uses' => 'CategoriesController@index',
  	'as' => 'categories'
  ]);
  Route::get('/categories/children', [
  	'uses' => 'CategoriesController@children',
  	'as' => 'categories.children'
  ]);
```

- Create Controller
```
php artisan make:controller CategoriesController
```

- Create methods in Controller
```
// ..
use App\Category;

class CategoriesController extends Controller
{
    public function index()
    {
    	$categories = Category::whereRaw('parent IS NULL')->pluck('name', 'id');
    	return view("categories.index", compact('categories'));
    }

    public function children(Request $request)
    {
    	return Category::where('parent', $request->parent)->pluck('name', 'id');
    }
}

```

- Create new view in "categories" directory and type this code:
```
@extends('layouts.app')

@section('content')
	<div class="form-group">
	     {!! Form::label('parent', 'Parent Category:')!!}
	     {!! Form::select('parent', $categories, null, ['placeholder' => 'Choose Category'])!!}
	</div>
	 
	<div class="form-group">
	     {!! Form::label('children', 'Child category:')!!}
	     {!! Form::select('children', [], null, ['placeholder' => 'Choose child category'])!!}
	</div>

@endsection

@section('form-script')
<script>
	$('#parent').change(function(e) {
		var parent = e.target.value;
		$.get('/categories/children?parent=' + parent, function(data) {
			$('#children').empty();
			$.each(data, function(key, value) {
				var option = $("<option></option>")
	                  .attr("value", key)		                  
	                  .text(value);

				$('#children').append(option);
			});
		});
	});
</script>
@endsection
```
