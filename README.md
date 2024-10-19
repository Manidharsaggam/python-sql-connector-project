from prettytable import PrettyTable
import mysql.connector as db

con = db.connect(username='root', password='add your password', host='localhost')
cur = con.cursor()

while True:
    print('-' * 6, 'Menu', '-' * 6)
    menu = '''1. Show databases
2. Create database
3. Use schema in database
4. Create table
5. Show tables
6. Data in the table
7. Insert data into table
8. Import data from file
9. Exit'''

    print(menu)
    ch = int(input('Choose your option in the menu: '))

    # SHOW DATABASES:
    if ch == 1:
        cur.execute('SHOW DATABASES')
        data = cur.fetchall()
        t = PrettyTable(['LIST OF DATABASES'])
        for i in data:
            t.add_row(i)
        print(t)
        print('*'*7,'ENd of Database List','*'*7)

    # CREATE DATABASE:
    elif ch == 2:
        d_name = input('Enter database name you want: ')
        cur.execute(f'CREATE DATABASE {d_name}')
        print('*'*7,'Database created','*'*7)

    # USE SCHEMA OR DATABASE:
    elif ch == 3:
        d_name = input('Enter your schema name: ')
        cur.execute(f'USE {d_name}')
        print('You are in', d_name, 'Schema')
        print('*'*7,'Connected','*'*7)

    # CREATING TABLE IN DATABASE:
    elif ch == 4:
        tables = int(input('Enter number of tables you want: '))
        for num in range(1, tables + 1):
            t_name = input('Enter table name: ')
            dict_v = {}
            col = int(input('Enter number of columns: '))
            for value in range(col):
                col_name = input('Enter name of column: ')
                data_type = input('Enter data type of column: ')
                dict_v[col_name] = data_type
            table = f"CREATE TABLE IF NOT EXISTS {t_name} ("
            for idx, (col_name, data_type) in enumerate(dict_v.items(), 1):
                if idx == col:
                    table += f'{col_name} {data_type}'
                else:
                    table += f'{col_name} {data_type}, '
            table += ')'
            cur.execute(table)
        print('*'*7,'Tables created','*'*7)

    # SHOW TABLES IN DATABASE:
    elif ch == 5:
        cur.execute('SHOW TABLES')
        data = cur.fetchall()
        t = PrettyTable(['LIST OF TABLES'])
        for i in data:
            t.add_row(i)
        print(t)
        print('*'*7,'Tables in the database','*'*7)

    # Data in the Table:
    elif ch == 6:
        table_name = input('Choose your table in the database: ')
        cur.execute(f'SELECT * FROM {table_name}')
        data = cur.fetchall()
        t = PrettyTable(['id', 'Name', 'Address'])
        for i in data:
            t.add_row(i)
        print(t)
        print('*'*7,'Showing data in the table','*'*7)

    # INSERT DATA INTO TABLE:
    elif ch == 7:
        tablename = input('Enter table name to which you want to insert: ')
        cur.execute(f'DESC {tablename}')

        m = len(cur.fetchall())
        no_records = int(input('Enter number of records: '))
        for _ in range(no_records):
            values = input('Enter values of records separated by comma: ')
            insert_query = f"INSERT INTO {tablename} VALUES ({','.join(['%s']*m)})" #m is len(data)
            cur.execute(insert_query, values.split(','))
            con.commit()
        print('*'*7,'Data inserted successfully','*'*7)


    # IMPORT DATA FROM FILE:
    elif ch == 8:
        # Prompt the user for file path, table name, and delimiters
        file_path = input('Enter file path here: ')
        table_name = input('Enter the table name: ')
        record_delimiter = input('Enter the record delimiter: ')
        column_delimiter = input('Enter the column delimiter: ')
        
        f = open(file_path, 'r')
        data = f.read().split(record_delimiter) 
        f.close()
        for record in data:
            cols = record.split(column_delimiter)
            if len(cols) > 1:
                cur.execute(f"DESCRIBE {table_name}")
                num_columns = len(cur.fetchall())
                if len(cols) == num_columns:
                    insert_query = f"INSERT INTO {table_name} VALUES ({','.join(['%s']*len(cols))})"
                    cur.execute(insert_query, cols)
        con.commit()
        print('*'*7,"Data imported successfully.",'*'*7)

    # EXIT FROM THE DATABASE:
    elif ch == 9:
        print('Exiting...')
        break
    else:
        print('Choose Correct Option')

con.commit()
cur.close()
con.close()
print('*'*7,'Disconnected','*'*7)
