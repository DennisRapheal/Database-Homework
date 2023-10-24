# 112-1 Database - HW1

[Set up Postgres on mac](https://www.notion.so/112-1-Database-HW1-e91b7e399bc44552a3038ff7bcbc7a16?pvs=21)

[Create database and tables](https://www.notion.so/112-1-Database-HW1-e91b7e399bc44552a3038ff7bcbc7a16?pvs=21)

[Queries](https://www.notion.so/112-1-Database-HW1-e91b7e399bc44552a3038ff7bcbc7a16?pvs=21)

[Output csv file link](https://www.notion.so/112-1-Database-HW1-e91b7e399bc44552a3038ff7bcbc7a16?pvs=21)

# Set up Postgres on mac

- Download Postgres

[https://postgresapp.com/downloads.html](https://postgresapp.com/downloads.html)

- First, log in Postgres with the port 5432 on  local host, than open `<database name>`

```bash
psql postgres://lintingwei@localhost:5432/<database name>
```

After entering the database, if  $<database name> = postgres$ ,we can see `posetgres=#`

# Create database & tables

1. The process of creating the database
2. The process of importing eight required .csv files into lego database

---

1. To create a database
    - Enter the Postgres database
    - create lego database
    
    ```sql
    create database lego;
    ```
    
    - Log in lego database
    
    ```bash
    psql postgres://lintingwei@localhost:5432/lego
    ```
    
    - Or use `\c lego` to change the user

1. Download the source .csv files from Kaggle.
    
    [LEGO Database](https://www.kaggle.com/datasets/rtatman/lego-database/data)
    
2. Import all .csv file, take colors.csv as example: 
    
    ```sql
    create table colors(
    	ID int,
    	name varchar(40),
    	rgb varchar(10),
    	is_trans char,
    	primary key(ID));
    ```
    
- To import this .csv file into a table, use `COPY` statement as follows:
    
    ```sql
    \copy colors from '/Users/lintingwei/Desktop/Database_HW/HW1_111550006/archive/colors.csv' delimiter ',' CSV HEADER;
    ```
    
    <aside>
    💡 Since we use the foreign key, we should import the build create the table as the followed sequence.
    
    </aside>
    

- import themes.csv
    
    ```sql
    create table themes(
    	ID int,
    	name varchar(40),
    	parent_id int,
    	primary key(ID));
    
    \copy themes from '/Users/lintingwei/Desktop/Database_HW/HW1_111550006/archive/themes.csv' delimiter ',' CSV HEADER;
    ```
    

- import sets.csv
    
    ```sql
    create table sets(
    	set_num varchar(30),
    	name varchar(100),
    	year int,
    	theme_id int,
    	num_parts int,
    	primary key(set_num),
    	foreign key(theme_id) references themes);
    
    \copy sets from '/Users/lintingwei/Desktop/Database_HW/HW1_111550006/archive/sets.csv' delimiter ',' CSV HEADER;
    ```
    

- import inventories.csv
    
    ```sql
    create table inventories(
    	ID int,
    	version int,
    	set_num varchar(30),
    	primary key(ID),
    	foreign key(set_num) references sets);
    
    \copy inventories from '/Users/lintingwei/Desktop/Database_HW/HW1_111550006/archive/inventories.csv' delimiter ',' CSV HEADER;
    ```
    

- import inventory_parts.csv
    
    ```sql
    create table inventory_parts(
    	inventory_id int,
    	part_num varchar(30),
    	color_id int,
    	quantity int,
    	is_spare char,
    	primary key(inventory_id, part_num, color_id, quantity, is_spare),
    	foreign key(inventory_id) references inventories(ID),
    	foreign key(color_id) references colors(ID));
    
    \copy inventory_parts from '/Users/lintingwei/Desktop/Database_HW/HW1_111550006/archive/inventory_parts.csv' delimiter ',' CSV HEADER;
    ```
    

- import inventory_sets.csv
    
    ```sql
    create table inventory_sets(
    	inventory_id int,
    	set_num varchar(30),
    	quantity int,
    	primary key(inventory_id, set_num),
    	foreign key(inventory_id) references inventories,
    	foreign key(set_num) references sets);
    
    \copy inventory_sets from '/Users/lintingwei/Desktop/Database_HW/HW1_111550006/archive/inventory_sets.csv' delimiter ',' CSV HEADER;
    ```
    

- import part_categories
    
    ```sql
    create table part_categories(
    	ID int,
    	name varchar(100),
    	primary key(ID));
    
    \copy part_categories from '/Users/lintingwei/Desktop/Database_HW/HW1_111550006/archive/part_categories.csv' delimiter ',' CSV HEADER;
    ```
    
- import parts.csv
    
    ```sql
    create table parts(
    	part_num varchar(20),
    	name varchar(500),
    	part_cat_id int,
    	primary key(part_num, name),
    	foreign key(part_cat_id) references part_categories);
    
    \copy parts from '/Users/lintingwei/Desktop/Database_HW/HW1_111550006/archive/parts.csv' delimiter ',' CSV HEADER;
    ```
    

- Tips( if you want to see the bottom of a table ):
    
    ```sql
    \pset pager on --關閉分頁
    ```
    

# Queries

1. ~ 8. The SQL statements and output results. 

---

- Query 4-a.
    - code:
    
    ```sql
    select s.name as set_name, t.name as theme_name, s.year
    from sets as s, themes as t
    where s.theme_id=t.id and s.year = 2017
    order by s.name;
    ```
    
    - output screen shot: (total: 296 rows)
    
    ![截圖 2023-10-24 下午2.22.01.png](112-1%20Database%20-%20HW1%20e91b7e399bc44552a3038ff7bcbc7a16/%25E6%2588%25AA%25E5%259C%2596_2023-10-24_%25E4%25B8%258B%25E5%258D%25882.22.01.png)
    

- Query 4-b.
    - code:
    
    ```sql
    select year, count(name) as total_number
    from sets
    group by year
    having year>=1950 and year<=2017
    order by total_number desc;
    ```
    
    - output screen shot: (total: 66 rows)
    
    ![截圖 2023-10-24 下午2.23.22.png](112-1%20Database%20-%20HW1%20e91b7e399bc44552a3038ff7bcbc7a16/%25E6%2588%25AA%25E5%259C%2596_2023-10-24_%25E4%25B8%258B%25E5%258D%25882.23.22.png)
    

- Query 4-c.
    - code:
    
    ```sql
    /*list the count of themes.name*/
    select themes.name as theme_name, count(sets.name) as popularity
    from sets join themes on sets.theme_id = themes.id
    group by themes.name
    order by popularity desc
    LIMIT 1;
    ```
    
    - output screen shot:
    
    ![截圖 2023-10-24 下午2.23.58.png](112-1%20Database%20-%20HW1%20e91b7e399bc44552a3038ff7bcbc7a16/%25E6%2588%25AA%25E5%259C%2596_2023-10-24_%25E4%25B8%258B%25E5%258D%25882.23.58.png)
    

- Query 4-d.
    - code:
    
    ```sql
    select themes.name as theme_name, avg(sets.num_parts) as avg_np
    from sets join themes on sets.theme_id = themes.id
    group by themes.name
    order by avg_np;
    ```
    
    - output screen shot: (total: 386 rows)
    
    ![截圖 2023-10-24 下午2.24.50.png](112-1%20Database%20-%20HW1%20e91b7e399bc44552a3038ff7bcbc7a16/%25E6%2588%25AA%25E5%259C%2596_2023-10-24_%25E4%25B8%258B%25E5%258D%25882.24.50.png)
    
- Query 4-e.
    - code:
    
    ```sql
    with temp_table(color, num_of_part) as(
    select colors.name as color, count(distinct parts.name) as num_of_part
    from  inventory_parts 
    			join colors on inventory_parts.color_id = colors.id
    			join parts on inventory_parts.part_num = parts.part_num
    group by colors.name)
    select color
    from temp_table
    order by num_of_part desc
    limit 10;
    
    ```
    
    - output screen shot:
    
    ![截圖 2023-10-24 下午2.25.26.png](112-1%20Database%20-%20HW1%20e91b7e399bc44552a3038ff7bcbc7a16/%25E6%2588%25AA%25E5%259C%2596_2023-10-24_%25E4%25B8%258B%25E5%258D%25882.25.26.png)
    

- Query 4-f.
    - code:
    
    ```sql
    with temp_table1(theme, color, quantity) as (
    select themes.name, colors.name, sum(inventory_parts.quantity)
    from  inventory_parts 
    			inner join colors on inventory_parts.color_id = colors.id
    			inner join inventories on inventory_parts.inventory_id = inventories.id
    			inner join sets on inventories.set_num = sets.set_num
    			inner join themes on sets.theme_id = themes.id
    group by themes.name, colors.name), 
    
    temp_table2(theme, max_num) as (
    select theme, max(quantity)
    from temp_table1 
    group by temp_table1.theme
    )
    
    select temp_table2.theme, temp_table2.max_num, temp_table1.color
    from temp_table2 join temp_table1 on temp_table1.theme = temp_table2.theme and temp_table1.quantity = temp_table2.max_num
    order by temp_table2.max_num;
    
    ```
    
    - output screen shot: (total: 384 rows)
    
    ![截圖 2023-10-24 下午2.27.48.png](112-1%20Database%20-%20HW1%20e91b7e399bc44552a3038ff7bcbc7a16/%25E6%2588%25AA%25E5%259C%2596_2023-10-24_%25E4%25B8%258B%25E5%258D%25882.27.48.png)
    

- Output the query to CSV file, the method is shown as followed
    
    ```sql
    \copy ( /*<query>*/ ) to '<path>' with delimiter ',' HEADER;
    ```
    
    <aside>
    💡 cannot include `‘\n’` after copy !
    
    </aside>
    
- Export query 4-a.
    
    ```sql
    \copy (select s.name as set_name, t.name as theme_name, s.year
    from sets as s, themes as t
    where s.theme_id=t.id and s.year = 2017
    order by s.name) to '/Users/lintingwei/Desktop/Database_HW/HW1_111550006/output/Query4_a.csv' with delimiter ',' HEADER;
    ```
    
- Export query 4-b.
    
    ```sql
    \copy (select year, count(name) as total_number from sets group by year having year>=1950 and year<=2017 order by total_number desc) to '/Users/lintingwei/Desktop/Database_HW/HW1_111550006/output/Query4_b.csv' with delimiter ',' HEADER;
    ```
    
- Export query 4-c.
    
    ```sql
    \copy (select themes.name as theme_name, count(sets.name) as popularity from sets join themes on sets.theme_id = themes.id group by themes.name order by popularity desc LIMIT 1) to '/Users/lintingwei/Desktop/Database_HW/HW1_111550006/output/Query4_c.csv' with delimiter ',' HEADER;
    ```
    
- Export query 4-d.
    
    ```sql
    \copy (select themes.name as theme_name, avg(sets.num_parts) as avg_np from sets join themes on sets.theme_id = themes.id group by themes.name order by avg_np) to '/Users/lintingwei/Desktop/Database_HW/HW1_111550006/output/Query4_d.csv' with delimiter ',' HEADER;
    ```
    
- Export query 4-e.
    
    ```sql
    \copy (with temp_table(color, num_of_part) as( select colors.name as color, count(distinct parts.name) as num_of_part from  inventory_parts  join colors on inventory_parts.color_id = colors.id join parts on inventory_parts.part_num = parts.part_num group by colors.name) select color from temp_table order by num_of_part desc limit 10) to '/Users/lintingwei/Desktop/Database_HW/HW1_111550006/output/Query4_e.csv' with delimiter ',' HEADER;
    ```
    
- Export query 4-f.
    
    ```sql
    \copy (with temp_table1(theme, color, quantity) as ( select themes.name, colors.name, sum(inventory_parts.quantity) from  inventory_parts  inner join colors on inventory_parts.color_id = colors.id inner join inventories on inventory_parts.inventory_id = inventories.id inner join sets on inventories.set_num = sets.set_num inner join themes on sets.theme_id = themes.id group by themes.name, colors.name),  temp_table2(theme, max_num) as (select theme, max(quantity) from temp_table1  group by temp_table1.theme) select temp_table2.theme, temp_table2.max_num, temp_table1.color from temp_table2 join temp_table1 on temp_table1.theme = temp_table2.theme and temp_table1.quantity = temp_table2.max_num order by temp_table2.max_num) to '/Users/lintingwei/Desktop/Database_HW/HW1_111550006/output/Query4_f.csv' with delimiter ',' HEADER;
    ```
    

# Output CSV Files Link

All  outputs of the queries are stored in the output file

[](https://github.com/DennisRapheal/Database-Homework/tree/c6fa08b1fd6af97a8ec3b399668e4e34412ad2a8/output)
