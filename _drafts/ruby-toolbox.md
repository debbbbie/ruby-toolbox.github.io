# ruby-toolbox #

## Active Record Plugins ##

### Active Record DB Adapters ###

### Active Record Default Values ###

- `default_value_for`
   <div class="description">定义 ActiveRecord models 的默认值</div>


        class User < ActiveRecord::Base
          default_value_for :name, "(no name)"
          default_value_for :last_seen {Time.now.to_s :db}
        end
        User.new.name # => "(no name)"
        User.new.last_seen # => 2013-12-07 09:58:43

- --`active_record_defaults`--
   <div class="description">类似 default_value_for ，N 久未更新</div>

### Active Record Enumerations ###

- `enumerize`

        class User
          extend Enumerize

          enumerize :sex, in: [:male, :female], predicates: true
        end
  ActiveRecord
  Mongoid
  MongoMapper
  I18n
  Predicate methods : `User.new.male? # => false`
  ActiveRecord scopes

- --`enumerated_attribute`--

   <div class="description">跟enumerize类似</div>

### Active Record Index Assistants ###

- `foreigner`
   <div class="description">migrations的外键支持</div>

        class Comment < ActiveRecord::Base
          belongs_to :post
        end

        class Post < ActiveRecord::Base
          has_many :comments, dependent: :delete_all
        end

        # 你需要在migration中加入外键
        add_foreign_key :comments, :posts, dependent: :delete
        # 或者这样写
        create_table :comments do |t|
          t.foreign_key :posts, dependent: :delete
        end

- `rails_indexes`

   <div class="description">帮助寻找忘加的索引</div>

        # 根据关系链，帮助寻找忘加或多加的索引
        rake db:index_migration

        # 根据代码中 find, find_by, find_by_x 等帮助寻找忘加或多加的索引
        rake db:find_query_indexes

- `schema_plus`

   <div class="description">ActiveRecord 增强版，更便捷增加索引、外键、默认值的写法等等</div>

        # 以前增加索引的写法
        create_table :parts do |t|
          t.string :name
        end
        add_index :parts, :name

        # SchemaPlus的写法
        create_table :parts do |t|
          t.string :name,  index: true
        end


        # 外键支持
        # 自动和表 'authors' 建立关系
        t.integer :author_id


        # 默认值
        # ActiveRecord::DB_DEFAULT 数据库默认值
        Post.create(category: ActiveRecord::DB_DEFAULT)

- --`immigrant`--

   <div class="description">类似`rails_indexes`， 根据models中的belong_to, has_many 等方法，计算出需要加的索引</div>

        rails generate immigration AddKeys

- --`lol_dba`

   <div class="description">`rails_indexes`和`migration_sql_generator`的合体</div>

### Active Record Named Scopes###
- `ransack`

   <div class="description">重新实现 MetaSearch ，一个方便搜索的 gem </div>

        # action这样写
        def index
          @q = Person.search(params[:q])
          @people = @q.result(distinct: true)
        end

        # view这样写
        # cont (contains)， 即搜索名字中包含某字的人
        <%= search_form_for @q do |f| %>
          <%= f.label :name_cont %>
          <%= f.text_field :name_cont %>
        <% end %>


- `searchlogic`

   <div class="description">利用 ActiveRecord 的 named scopes，扩展了很多好用的方法</div>


        # 比如我们有这样的model
        User(:id, :created_at, :username, :age)

        # SearchLogic 支持以下的方法
        User.username_equals("debbbbie")
        User.username_like("debbbbie")
        User.username_begins_with("debbbbie")
        User.username_ends_with("debbbbie")
        User.age_greater_than(20)
        User.username_null
        User.username_blank
        User.username_like_any("debbbbie", "ian")

        User.username_not_xxx

        # 更短的别名方法
        User.username_is("debbbbie")
        User.username_eq("debbbbie")
        User.age_gt(20)

- `pacecar`

   <div class="description">来自thoughtbot，类似searchlogic，为 ActiveRecord 扩展了许多方法</div>
        User.username_present
        User.username_missing
        User.by_username      # order
        User.username_equals("debbbbie")

        # 状态
        User.login_state_login
        User.login_not_login

- `randumb`

   <div class="description">从数据库随机的读取记录</div>

        Artical.random    # 随即取一篇文章
        Artical.randow(3) # 随即取三篇文章

- --`can_search`--
- --`utility_scopes`--

### Active Record Nesting ###

- `awesome_nested_set`

   <div class="description">替代acts_as_nested_set和better_nested_set</div>


        # 表中需要 :lft, :rgt, :parent_id 等字段
        create_table :categories do |t|
          t.integer :name
          t.integer :parent_id
          t.integer :lft
          t.integer :rgt
        end

        # model
        class Category < ActiveRecord::Base
          acts_as_nested_set
        end

        # methods
        root?
        child?
        # ...
- `ancestry`

   <div class="description">ActiveRecord的树状结构，优点比acts_as_tree多</div>

        class Category < ActiveRecord::Base
          act_as_tree
        end

        # methods
        parent,  root, root?, ancestors, children, path, has_children, siblings, descendants, subtree, depth ...

- `acts_as_tree`

   <div class="description">rails官方支持，但N久未更新，可以考虑用ancestry</div>

- `closure_tree`

   <div class="description">比ancestry和acts_as_tree效率更高，比awesome_nested_set更优秀，还支持生成树状图</div>

### Active Record Sharding ###
- `ar-octopus`

   <div class="description">很好的支持Database Sharding。比DbCharmer, DataFabric, MultiDb都好</div>
        # models
        User.limt(3).using(:slave_one)

        Octopus.using(:slave_two) do
          User.create(name: "Mike")
        end

        # migrations
        class CreateUsersOnBothShards < ActiveRecord::Migration
          using(:shard1, :shard2)
          def self.up
            User.create!(name: "Both")
          end
        end

        # controllers
        class ApplicationController < ActiveController::Base
          around_filter :select_shard

          def select_shard(&block)
            Octopus.using(:shard1, &block)
          end
        end

- `db-charmer`

   <div class="description">看文档好复杂的样子，感觉比ar-octopus牛叉啊，等深入研究看看</div>

### Active Record Soft Delete ###

- `rails3_acts_as_paranoid`

   <div class="description">软删除，可以恢复</div>

        class Artical < ActiveRecord::Base
          acts_as_paranoid
        end

        # filter
        Artical.only_deleted # 已删除的文章
        Artical.with_deleted # 所有文章，包含已删除的
        Artical.deleted_before_time(time)# 多久时间之前删除的文章

        # delete
        artical.destroy! # true delete
        artical.destroy  # soft delete

        # recovery
        Artical.only_deleted.where("name=?","debbbbie").first.recover

        # validation
        class Artical < ActiveRecord::Base
          acts_as_paranoid
          validates_as_paranoid
          validates_uniqueness_of_without_deleted :name
        end
        a1 = Artical.create(name: "haha")
        a1.destroy
        a2 = Artical.create(name: "haha")
        a2.valid? # => true
        a2.save
        a1.revover # => validate failed!


### Active Record Sortables ###

- `acts_as_list`

    <div class="description">ActiveRecord支持排序</div>

        class TodoList < ActiveRecord::Base
          has_many :todo_items, -> { order("position DESC") }
        end
        class TodoItem < ActiveRecord::Base
          belongs_to :todo_list
          acts_as_list scope: :todo_list
        end

        todo_list.first.move_to_bottom
        todo_list.last.move_higher

        todo_item.insert_at(2)
        todo_item.mover_lower
        todo_item.increment_position
        todo_item.first?
        todo_item.higher_item

### Active Record User Stamping ###


- `userstamp`

### Benchamark ###

- `benchmark`

   <div class="description">Ruby自带的类库。</div>
  [Benchmark计算ruby代码执行时间](http://www.51testing.com/html/93/n-827893.html)
  [http://blog.itpub.net/442622/viewspace-824734/](http://blog.itpub.net/442622/viewspace-824734/)
  [http://www.codesky.net/article/200910/166589.html](http://www.codesky.net/article/200910/166589.html)
  [http://www.ibm.com/developerworks/cn/opensource/os-railsn1/](http://www.ibm.com/developerworks/cn/opensource/os-railsn1/)

- `method_profiler`

   <div class="description">收集对象的方法并统计运行的时间，最终给出报表，方便找出运行慢的方法</div>


### cli_option_parsers ###
 已提交至 github

### cli_progress_bars ###

- `ruby-progressbar`

   <div class="descrition">Ruby的进度条</div>
		require 'ruby-progressbar'
		ProgressBar.create
		50.times { sleep 1; p.increment }
		Progress: |===================================                                    |
- `progressbar`

   <div class="description">那个o可以改改吗。。</div>
		pbar = ProgressBar.new("test", 100)
		=> (ProgressBar: 0/100)
		>> (1..100).each{|x| sleep(0.1); pbar.set(x)}; pbar.finish
		test:           67% |oooooooooooooooooooooooooo              | ETA:  00:00:03
- `progress_bar`
		require 'progress_bar'
		bar = ProgressBar.new

		100.times do
		  sleep 0.1
		  bar.increment!
		end
		[#######################################                           ] [ 59.00%] [00:06]

### 词汇表 ###

---
predicate 断言的
nested_set 嵌套集合（比如二叉树）
---
