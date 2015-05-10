# Model

## 前置条件
使用此类之前必须要加载 PDO类库，设置好数据库链接等。   

	$vitex->using(new \Vitex\Ext\Pdo([
	'host'     => 'localhost',
	'database' => 'test',
	'charset'  => 'utf8',
	], 'root', 'root'));  
第一个参数 可以是一个数组，包含了 host、database、charset信息，也可以是一个 PDO DSN的链接字符串，还可以是一个PDO的链接对象（此时无需指定后2个参数）。

第二个参数为用户名   
第三个参数为密码   

	$vitex->using(new \Vitex\Ext\Pdo(’mysql:dbname=test;host=localhost;charset=utf8‘, 'root', 'root'));


Model是一个简单的ORM，非常轻量级的数据库操作类。
## 使用
	use \Vitex\Ext\Model;

	class User extends Model
	{

	}
user 表存在三个字段  id  name age;   
如上声明了一个继承自Model的user类，默认情况下会按把类名（小写）当做表名来查询数据，默认的主键为`id`，如上例子：
	
	$user = new User();  

	//简单查询   
	$user->get(1); // select * from user where id = 1  
	
	//  根据上一个查询结果直接修改值  	
	$user->name = "Vitex";
	$user->save(); // update user set name = 'Vitex' where id = 1;  // 会自动调用 get方法设置的主键ID进行修改
	// 按照条件查找指定字段内容  
	$user->select('id')->where('age','>',18)->getAll(); // select id from user where age > 18  
	
	//子查询  
	$user->whereIn('age',Model::sub()->from('user')->select(age)->where('id', '>', 10))->getAll();  
	select * from user where age in (select age from user where id > 10)  

[更多示例](Ext.Model.Example.html)    

## API 

###def()
定义一个新的模型数据，也就是说初始化一条记录，这条记录的字段应该是与数据库对应的,支持链式操作        
**签名**  
`def(array  $arr = array()) `  
**参数**  
array 	$arr 	数据数组，字段对应数据库中表的字段  
**示例**  
`$model->def(['name'=>'Vitex','age'=>26])`

###setPrefix()
设置表前缀,此方法直接返回对象本身支持链式操作,支持链式操作      
**签名**  
`setPrefix(string  $prefix) : object`  
**参数**  
string 	$prefix 前缀
**示例**  
`$model->setPrefix('cms_')`  

###sub()
返回当前对象的实例，此方法为一个静态方法会返回新实例化的Model类，一般用于子查询实例化model,支持链式操作    
**签名**  
`sub() : object`  
**示例**  
`\Vitex\Ext\Model::sub()->from('user')->select('id')`  
//如果当做条件传递给`where`会自动调用toString方法转为字符串 `select id from user `  

###query()
直接执行sql语句 @#_ 当做表前缀替换掉    
**签名**  
`query(string  $sql) : mixed`  
**参数**  
string 	$sql 	sql语句
**示例**  
`$model->query("select * from @#_user") ` // select * from cms_user  

###select()
设置要查询的字段名,支持链式操作    
**签名**  
`select(mixed  $column = '*') : object`  
**参数**  
mixed 	$column 可以是字符串，多个字段用,分开，也可以是数组每个元素为一个字段，也可以是*  
**示例**  
`$model->select("*")`  
`$this->select('id,name')`  
`$this->select(['name','id'])`  

###whereRaw()
字符串形式的查询语句,使用该方法一定要了解你在做什么，此方法设置的条件总是会在where设置的条件之后，
也就是说如果你使用where设置了条件那么你使用本方法设置时应该注意前面添加  and/or 等连接符,支持链式操作          
**签名**  
`whereRaw(string  $val) : \Vitex\Ext\object`  
**参数**  
string 	$val 	查询条件语句  
**示例**  
`$this->whereRaw("`name`='Vitex'")`
`$this->where('age','>',26)->whereRaw("and `name`='Vitex'")`  

>**注意** 下面 where系列的方法 默认都是以`and` 连接不同的条件，orWhere系列的方法默认都是用 `or`连接不同的条件。  

###where /orWhere
设置查询条件 where语句   
**签名**   
`where(string  $key,string $op,string $val) : \Vitex\Ext\object`   
**参数**   
string $key 键值，字段名   
string $op 操作符 如  = > < >= <= 等   
string $val 值
**示例**   
`$this->where('id','=',1)`  

###whereIn /orWhereIn
查询语句  where in 语句   
**签名**   
`whereIn(string  $key,string $val) : \Vitex\Ext\object`   
**参数**   
string $key 键值，字段名   
string/array/object $val 值
**示例**   
`$this->whereIn("name","a,b,c")` where name in ('a','b','c')  
`$this->whereIn("name",['a','b',c])` // 同上  
`$this->whereIn('id',\Vitex\Ext\Model::sub()->from("user")->select('id')) `  
where id in (select id from user)
//如果是子查询的whereIn，请确保子查询的代码中不会包含 `,`,如果包含 `,`可能会导致错误   

###whereNotIn / orWhereNotIn
查询语句  where not in 语句   
**签名**   
`whereNotIn(string  $key,string $val) : \Vitex\Ext\object`   
**参数**   
string $key 键值，字段名   
string/array/object $val 值
**示例**   
`$this->whereNotIn("name","a,b,c")` where name not in ('a','b','c')  
`$this->whereNotIn("name",['a','b',c])` // 同上  
`$this->whereNotIn('id',\Vitex\Ext\Model::sub()->from("user")->select('id')) `  
where id not in (select id from user)
//如果是子查询的whereNotIn，请确保子查询的代码中不会包含 `,`,如果包含 `,`可能会导致错误   

###whereNull  / orWhereNull
查询语句 is null   
**签名**   
`whereNull(string  $key) : \Vitex\Ext\object`   
**参数**   
string $key 键值，字段名   
**示例**   
`$this->whereNull('name')` // where name is null  

###whereNotNull / orWhereNotNull 
查询语句 is not null   
**签名**   
`whereNotNull(string  $key) : \Vitex\Ext\object`   
**参数**   
string $key 键值，字段名   
string/array/object $val 值   
**示例**   
`$this->whereNotNull('name')` // where name is not null 

###whereExists / orWhereExists
查询语句 EXISTS    
**签名**   
`whereExists(object/string  $key) : \Vitex\Ext\object`   
**参数**   
string $key 子查询   
**示例**   
`$this->whereExists(\Vitex\Ext\Model::sub()->from("user")->select('id,name')) `   
//where exists (select id,name from user)   
`$this->whereExists('select id,name from user') `     

###whereNotExists / orWhereNotExists 
查询语句 NOT EXISTS    
**签名**   
`whereNotExists(object/string  $key) : \Vitex\Ext\object`   
**参数**   
string $key 子查询   
**示例**   
`$this->whereNotExists(\Vitex\Ext\Model::sub()->from("user")->select('id,name')) `   
//where not exists (select id,name from user)   
`$this->whereNotExists('select id,name from user') `     

###whereBetween / orWhereBetween
操作符 BETWEEN ... AND 会选取介于两个值之间的数据范围。这些值可以是数值、文本或者日期。   
**签名**   
`whereBetween(string  $key,array $val) : \Vitex\Ext\object`   
**参数**   
string $key 键值，字段名   
array $val 值,一个包含两个元素的数组， between ele1 and ele2   
**示例**   
`$this->whereBetween('age',[10,20])` between 10 and 20   

###whereNotBetween / orWhereNotBetween
操作符 Not BETWEEN ... AND 会排除介于两个值之间的数据范围。这些值可以是数值、文本或者日期。   
**签名**   
`whereNotBetween(string  $key,array $val) : \Vitex\Ext\object`   
**参数**   
string $key 键值，字段名   
array $val 值,一个包含两个元素的数组， not between ele1 and ele2   
**示例**   
`$this->whereNotBetween('age',[10,20])` not between 10 and 20   


###having()
Having分组操作条件,支持链式操作    
**签名**  
`having(string  $key, string $op, array/callable  $val, string  $type = "AND") : object`  
**参数**  
string 	$key 	键值
string 	$op 	操作符
array/callable 	$val 	操作值
string 	$type 	类型 and/or  
**示例**  
`$this->having('num','>',100,'and')`  

###from()
要查询的表名,支持链式操作    
**签名**  
`from(string  $table) : object`  
**参数**  
string 	$table 	表名  
**示例**  
`$this->from('user')`  

###limit()
查询的条数,支持链式操作         
**签名**  
`limit(string  $limit, integer  $offset) : object`  
**参数**  
string 	$limit 	要查询的条数
integer 	$offset 	偏移值 默认0
**示例**  
`$this->limit(10,2)` // limit 2,10   
`$this->limit(10)`  // limit 10  

###getTable()
获取当前要查询的表名    
**签名**  
`getTable() : string`  

**示例**  

`$table = $this->getTable()`  


###offset()
设置查询的偏移数制,支持链式操作         
**签名**  
`offset(integer  $offset) : object`  
**参数**  
integer 	$offset 	偏移数值  
**示例**  
`$this->limit(10)->offset(4)` // limit 4,10  


###orderBy()
设置排序字段以及排序方式,支持链式操作       
**签名**  
`orderBy(string  $column, string  $way = "DESC") : object`  
**参数**  
string 	$column 	字段
string 	$way 	排序方式  
**示例**  
`$this->orderBy('age','desc')` == `$this->orderBy('age')`  

###groupBy()
group分组操作,支持链式操作        
**签名**  
`groupBy(string  $column) : object`  
**参数**  
string 	$column 	要分组的字段  
**示例**  
`$this->groupBy('name')`  

###distinct()
去重查询,支持链式操作        
**签名**  
`distinct(string/array  $column) : object`
**参数**  
string/array 	$column 	字段名  
**示例**  
`$this->distinct('name')` // select distinct name;  
`$this->distinct(['name','age'])` // select distinct name,age  

###union()
union操作连表查询        
**签名**  
`union(string/callable  $str) : object`  
**参数**  
string/callable 	$str 	union字符串或者一个可以tostring的对象（例如model对象的实例）  
**示例**  
`$this->union('select * from user')`  
`$this->union(\Vitex\Ext\Model::sub()->from('user'))`  

###set()
修改查询的数据 设置要保存的数据，调用`save`方法时会使用此方法设置的数据      
**签名**  
`set(string  $key, string  $val) : object`  
**参数**  
string/array 	$key 	键值  
string 	$val 	值  
**示例**  
`$this->set('name','Vitex')->save(1)` // 根据主键作为条件保存数据,明确指定主键  
`$this->get(1);$this->set('name','Vitex')->save()` //使用get方法获取的数据的主键  
`$this->set(['name'=>'vitex','age'=>10]).save()`

###update()
根据where条件修改内容      
**签名**  
`update(array  $arr) : mixed`  
**参数**  
array $arr 要修改的数据 关联数组，键名为数据库字段名    
**示例**  
`$this->from('user')->where('id','=',1)->update(['name'=>'Vitex'])`  

###insert()
向数据库中插入数据，可以是多维数组； 当为二维数组的时候插入多条数据        
**签名**  
`insert(array  $arr = array()) : mixed`  
**参数**  
array 	$arr 关联数组，一维或者二维数组，键值为数据库字段名  
**示例**  
`$this->insert(['name'=>'Vitex'])`  
`$this->insert([['name'=>'Vitex1'],['name'=>'Vitex2']])`  
//表名默认为类名（小写）

###save()
ORM似的保存 保存当前模型，如果存在主键则尝试修改，如果不存在主键则尝试新建  
**注意** 如果设定了排除字段则设定的排除字段不参与修改    

**签名**  
save(mixed $id) : mixed  
**参数**  
mixed $id  主键的值，保存时的条件，新增加的数据不需要指定该字段   
**示例**  
`$this->name = 'Vitex'; $this->save();`  insert into user (`name`) values ('Vitex');  

###delete()
删除数据       
**签名**  
`delete() : boolean`    

**示例**
`$this->where('id','=',1)->detele()`  

###truncate()
清空指定表中的数据       
**签名**  
`truncate() : object`  
**示例**  
`$this->truncate()` // 默认表  
`$this->from('user')->truncate()`   

###increment()
自增一个字段的值       
**签名**  
`increment(string  $column, integer  $amount = 1) : boolean`  
**参数**  
string 	$column 字段名  
integer $amount  自增的数制默认为1   
**示例**  
`$this->increment('pv',1)`  
`$this->from('table')->increment('click',3)`  

###decrement()
自减一个字段的值       
**签名**  
`decrement(string  $column, integer  $amount = 1) : boolean`   
**参数**  
string 	$column 	字段名   
integer $amount 自增的数制默认为1   
**示例**  
`$this->decrement('pv',1)`  
`$this->from('table')->decrement('click',3)` 

###count()
统计数量，select count(*) from user            
**签名**  
`count(string  $column = '*') : integer`    
**参数**  
`string $column 字段名`    
**示例**  
`$this->count()` // select count(*) from user   
`$this->from('table')->count('name')`  

###pluck()
一个简化的array_map操作，可以按照指定的字段返回一个仅包含该字段的数组    
**签名**  
`pluck(string  $column) : array`   
**参数**  
string 	$column 字段名
**示例**  
`$this->from('user')->pluck('name')`  
返回一个name组成的数组 ['Vitex1','Vitex2']

###get()
根据主键获取一条记录       
**签名**  
`get(string  $id = null) : mixed`  
**参数**  
string 	$id ID主键值   
**示例**  
`$this->get(1)` // select * from user where id=1   

> getBy.. 是一个系列方法，如果您的数据表中包含指定的字段那么就可以直接使用该方法获取指定字段的内容  

### getBy..
本系列方法会把 `By`后面的内容转为小写然后当做数据库的字段来查询,返回符合条件的单条数据
**示例**  

`$this->getByName('vitex')` // select * from table where name = 'vitex'   
`$this->getByAge(10)` // select * from table where age = 10   



###getAll()
根据查询条件返回数组       
**签名**  
`getAll() : array`   
**示例**  
`$this->where('age','>','18')->getAll()` select * from user where age>18   

> getAllBy.. 是一个系列方法，如果您的数据表中包含指定的字段那么就可以直接使用该方法获取指定字段的内容  

### getAllBy..
本系列方法会把 `By`后面的内容转为小写然后当做数据库的字段来查询,返回符合条件的多条或者单条数据，为一个二维数组   
**示例**  

`$this->getAllByName('vitex')` // select * from table where name = 'vitex'   
`$this->getAllByAge(10)` // select * from table where age = 10   


###page()
直接按照分页查询相关的信息，包括总页数以及当前分页的内容       
**签名**  
`page( integer $page = 1, integer  $num = 10) : array`   
**参数**  
integer 	$page  页码 默认为1 	  
integer 	$num   每页条数，默认为10  
**返回值**   
[infos,total]  
第一个元素是查询出来的信息具体内容  
第二个元素是当前查询条件下的总条数  
**示例**  
`$this->where('age','>',10)->page()` // select * from user where age > 1 limit 0,10   
`list($lists,$total) = $this->page(10,10)` // select * from user limit 90,10
