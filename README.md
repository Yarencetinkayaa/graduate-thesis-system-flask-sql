# graduate-thesis-system-flask-sql
A web-based graduate thesis management system using Python Flask and Microsoft SQL Server.
# Graduate Thesis Management System (GTS)

This is a web-based application for managing graduate thesis records. It is built with **Python (Flask)** and **Microsoft SQL Server**. Users can add, edit, delete, and search thesis data, as well as manage universities, institutes, authors, supervisors, and keywords.

# Technologies Used

- Python (Flask)
- Microsoft SQL Server
- pyodbc (Database connection)
- HTML, CSS (Templates)
- Bootstrap (for responsive design)

# Features

- Add and manage thesis records
- Associate theses with authors, supervisors, topics, and keywords
- University and institute management
- Advanced search functionality
- CRUD operations (Create, Read, Update, Delete)
- SQL triggers for integrity control

# How to Run

Clone the repo:
```bash
git clone https://github.com/yourusername/graduate-thesis-system.git
cd graduate-thesis-system

 All source code is in app.py

from flask import Flask, request, render_template, redirect, url_for, flash
import pyodbc

app = Flask(__name__)

app.secret_key = 'yarensecretkey' 

def get_db_connection():
    conn = pyodbc.connect(
        r'DRIVER={ODBC Driver 17 for SQL Server};'
        r'SERVER=DESKTOP-V9R50E7\SQLEXPRESS;' 
        r'DATABASE=Graduate_Thesis_System;'  
        r'Trusted_Connection=yes;'
    )
    return conn

# Home page and listing all thesis
@app.route('/')
def index():
    conn = get_db_connection()
    cursor = conn.cursor()
    query = "SELECT * FROM Thesis"
    cursor.execute(query)
    results = cursor.fetchall()
    conn.close()
    return render_template('index.html', theses=results)

#Add new thesis page
@app.route('/add_keyword_and_thesis', methods=['GET', 'POST'])
def add_keyword_and_thesis():
    if request.method == 'POST':
        conn = None  
        try:
            
            keyword_id = request.form.get('keyword_id')
            keyword = request.form.get('keyword')
            topic_id = request.form.get('topic_id')
            topic_name = request.form.get('topic_name')
            thesis_no = request.form.get('thesis_no')
            title = request.form.get('title')
            abstract = request.form.get('abstract')
            year = request.form.get('year')
            thesis_type = request.form.get('type')
            language = request.form.get('language')
            university_id = request.form.get('university_id')
            institute_id = request.form.get('institute_id')
            pages_no = request.form.get('pages_no')
            author_id = request.form.get('author_id')
            submission_date = request.form.get('submission_date')

            print("Formdan alınan veriler:")
            print(f"KeywordID: {keyword_id}, Keyword: {keyword}")
            print(f"ThesisNo: {thesis_no}, Title: {title}, Abstract: {abstract}, Year: {year}, Type: {thesis_type}")
            print(f"Language: {language}, UniversityID: {university_id}, InstituteID: {institute_id}, PagesNo: {pages_no}")
            print(f"AuthorID: {author_id}, SubmissionDate: {submission_date}")

            supported_languages = ['English', 'Turkish', 'French']

       
            if language not in supported_languages:
                flash(f"Invalid language: {language}. Accepted values are: {', '.join(supported_languages)}", 'danger')
                return redirect(url_for('add_keyword_and_thesis'))

           
            conn = get_db_connection()
            cursor = conn.cursor()

      
            conn.autocommit = False

           
            query_keywords = """
            INSERT INTO Keywords (KeywordID, Keyword)
            VALUES (?, ?)
            """
            cursor.execute(query_keywords, (keyword_id, keyword))
            print(f"Keywords tablosuna eklendi: KeywordID={keyword_id}, Keyword={keyword}")

            
            query_subject_topics = """
            INSERT INTO SubjectTopics (TopicID, TopicName)
            VALUES (?, ?)
            """
            print("SubjectTopics tablosuna ekleme yapılıyor...")
            cursor.execute(query_subject_topics, (topic_id, topic_name))
            print(f"SubjectTopics tablosuna eklendi: TopicID={topic_id}, TopicName={topic_name}")

        
            query_thesis_topic = """
            INSERT INTO ThesisTopic (TopicID, ThesisNo)
            VALUES (?, ?)
            """
            print("ThesisTopic tablosuna ekleme yapılıyor...")
            cursor.execute(query_thesis_topic, (topic_id, thesis_no))
            print(f"ThesisTopic tablosuna eklendi: TopicID={topic_id}, ThesisNo={thesis_no}")

           
            query_thesis_keyword = """
            INSERT INTO ThesisKeyword (ThesisNo, KeywordID)
            VALUES (?, ?)
            """
            print("ThesisKeyword tablosuna ekleme yapılıyor...")
            cursor.execute(query_thesis_keyword, (thesis_no, keyword_id))

            # Add data to the Thesis table
            query_thesis = """
            INSERT INTO Thesis (ThesisNo, Title, Abstract, Year, Type, Language, UniversityID, InstituteID, PagesNo, AuthorID, SubmissionDate)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
            """
            print("Thesis tablosuna ekleme yapılıyor...")
            cursor.execute(query_thesis, (thesis_no, title, abstract, year, thesis_type, language, university_id, institute_id, pages_no, author_id, submission_date))

           
            print("Veritabanı commit ediliyor...")
            conn.commit()
            flash('Keyword and Thesis added successfully!', 'success')

        except Exception as e:
            print(f"Hata oluştu: {str(e)}")
            if conn:
                conn.rollback() 
            flash(f'Error: {str(e)}', 'danger')

        finally:
            print("Veritabanı bağlantısı kapatılıyor...")
            if conn:
                conn.close() 

        return redirect(url_for('index'))

 
    return render_template('add_keyword_and_thesis.html')



#New person adding page
@app.route('/add_person_and_supervisor', methods=['GET', 'POST'])
def add_person_and_supervisor():
    if request.method == 'POST':
        person_id = request.form.get('person_id')
        name = request.form.get('name')
        title = request.form.get('title')
        thesis_no = request.form.get('thesis_no')

        try:
            conn = get_db_connection()
            cursor = conn.cursor()

            
            conn.autocommit = False

           
            query_persons = "INSERT INTO Persons (PersonID, Name, Title) VALUES (?, ?, ?)"
            cursor.execute(query_persons, (person_id, name, title))

            
            query_supervisor = "INSERT INTO Supervisor (PersonID, ThesisNo) VALUES (?, ?)"
            cursor.execute(query_supervisor, (person_id, thesis_no))

            
            conn.commit()
            conn.close()

           
            flash('Person and Supervisor added successfully!', 'success')

        except Exception as e:
            conn.rollback() 
            conn.close()
            flash(f'Error: {str(e)}', 'danger')
            return redirect(url_for('add_person_and_supervisor'))

    return render_template('add_person_and_supervisor.html')

@app.route('/add_university', methods=['GET', 'POST'])
def add_university():
    if request.method == 'POST':
        university_id = request.form.get('university_id')
        name = request.form.get('name')

        try:
            conn = get_db_connection()
            cursor = conn.cursor()

           
            query_university = "INSERT INTO Universities (UniversityID, Name) VALUES (?, ?)"
            cursor.execute(query_university, (university_id, name))

            #
            conn.commit()
            flash('University added successfully!', 'success')

        except Exception as e:
            conn.rollback()  
            flash(f'Error: {str(e)}', 'danger')

        finally:
            conn.close() 

        

   
    return render_template('add_university.html')

#New institute adding page
@app.route('/add_institute', methods=['GET', 'POST'])
def add_institute():
    if request.method == 'POST':
        institute_id = request.form.get('institute_id')
        university_id = request.form.get('university_id')
        name = request.form.get('name')

        try:
            conn = get_db_connection()
            cursor = conn.cursor()

           
            query_institute = """
            INSERT INTO Institutes (InstituteID, UniversityID, Name)
            VALUES (?, ?, ?)
            """
            cursor.execute(query_institute, (institute_id, university_id, name))

            
            conn.commit()
            conn.close()

       
            flash('Institute added successfully!', 'success')

        except Exception as e:
            conn.rollback()  
            conn.close()
            flash(f'Error: {str(e)}', 'danger')
            return redirect(url_for('add_institute'))

    return render_template('add_institute.html')

@app.route('/view/<table_name>')
def view_table(table_name):
    try:
        conn = get_db_connection()
        cursor = conn.cursor()
        query = f"SELECT * FROM {table_name}"
        cursor.execute(query)
        columns = [column[0] for column in cursor.description]
        rows = cursor.fetchall()
        return render_template('view_table.html', table_name=table_name, columns=columns, rows=rows)
    except Exception as e:
        flash(f'Error fetching data: {str(e)}', 'danger')
        return redirect(url_for('index'))
    finally:
        conn.close()

@app.context_processor
def utility_processor():
    return dict(zip=zip)

#Editing tables page
@app.route('/edit/<table_name>/<int:id>', methods=['GET', 'POST'])
def edit_table(table_name, id):
    try:
        conn = get_db_connection()
        cursor = conn.cursor()

        
        id_column = (
            'UniversityID' if table_name.lower() == 'universities' else
            'TopicID' if table_name.lower() == 'subjecttopics' else
            'InstituteID' if table_name.lower() == 'institutes' else
            'ID'
        )

        if request.method == 'POST':
            form_data = request.form.to_dict()
            updates = ", ".join([f"{key} = ?" for key in form_data.keys()])
            values = list(form_data.values()) + [id]

            query = f"UPDATE {table_name} SET {updates} WHERE {id_column} = ?"
            cursor.execute(query, values)
            conn.commit()
            flash(f'Record updated in {table_name}!', 'success')
            return redirect(url_for('view_table', table_name=table_name))

        query = f"SELECT * FROM {table_name} WHERE {id_column} = ?"
        cursor.execute(query, (id,))
        record = cursor.fetchone()
        columns = [column[0] for column in cursor.description]
        return render_template('edit_table.html', table_name=table_name, columns=columns, record=record)

    except Exception as e:
        flash(f'Error: {str(e)}', 'danger')
        return redirect(url_for('view_table', table_name=table_name))
    finally:
        conn.close()

@app.route('/delete/<table_name>/<int:id>', methods=['POST'])
def delete_record(table_name, id):
    try:
        conn = get_db_connection()
        cursor = conn.cursor()

       
        primary_key_columns = {
            'universities': 'UniversityID',
            'subjecttopics': 'TopicID',
            'institutes': 'InstituteID',
            'persons': 'PersonID', 
        }

       
        id_column = primary_key_columns.get(table_name.lower(), 'ID')

        
        query = f"DELETE FROM {table_name} WHERE {id_column} = ?"
        cursor.execute(query, (id,))
        conn.commit()
        flash(f'Record deleted from {table_name}!', 'success')

    except Exception as e:
        flash(f'Error deleting record: {str(e)}', 'danger')
    finally:
    
        if 'conn' in locals():
            conn.close()

    return redirect(url_for('view_table', table_name=table_name))

#Thesis search page with each data
@app.route('/search', methods=['GET', 'POST'])
def search():
    results = []
    if request.method == 'POST':
        column_name = request.form.get('column_name')
        search_value = request.form.get('search_value')

        try:
            conn = get_db_connection()
            cursor = conn.cursor()

           
            query = f"SELECT * FROM Thesis WHERE {column_name} LIKE ?"
            cursor.execute(query, ('%' + search_value + '%',))
            results = cursor.fetchall()

            conn.close()
        except Exception as e:
            flash(f"Error: {str(e)}", 'danger')

    return render_template('search.html', results=results)




if __name__ == '__main__':
    app.run(debug=True)

