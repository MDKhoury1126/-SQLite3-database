# -SQLite3-database
``` import sqlite3

# ----------------------------------------
# 1. Connect to the database (or create it)
# ----------------------------------------
conn = sqlite3.connect('orgdb.sqlite')
cur = conn.cursor()

# ----------------------------------------
# 2. Reset the table so each run starts clean
# ----------------------------------------
cur.execute('DROP TABLE IF EXISTS Counts')

cur.execute('''
    CREATE TABLE Counts (
        org   TEXT,
        count INTEGER
    )
''')

# ----------------------------------------
# 3. Open the mailbox file
# ----------------------------------------
fname = input('Enter file name: ')
if len(fname) < 1:
    fname = 'mbox.txt'

fh = open(fname)

# ----------------------------------------
# 4. Process the file line by line
# ----------------------------------------
for line in fh:
    if not line.startswith('From: '):
        continue

    pieces = line.split()
    email = pieces[1]              # full email, e.g. 'cwen@iupui.edu'

    # Extract the organization (domain) part
    org = email.split('@')[1]      # 'iupui.edu'

    # ------------------------------------
    # 5. Check if this org is already in DB
    # ------------------------------------
    cur.execute('SELECT count FROM Counts WHERE org = ?',
                (org,))
    # “Find the row where the SQL column org equals the value I will supply.”
    row = cur.fetchone()

    # ------------------------------------
    # 6. Insert new org or update existing
    # ------------------------------------
    if row is None:
        cur.execute('INSERT INTO Counts (org, count) VALUES (?, 1)',
                    (org,))
    else:
        cur.execute('UPDATE Counts SET count = count + 1 WHERE org = ?',
                    (org,))

# ----------------------------------------
# 7. Commit once after the loop (faster)
# ----------------------------------------
conn.commit()

# (Optional) Show top organizations
sqlstr = '''
    SELECT org, count
    FROM Counts
    ORDER BY count DESC
    LIMIT 10
'''

print('\nTop organizations:\n')
for row in cur.execute(sqlstr):
    print(row[0], row[1])

cur.close()```
