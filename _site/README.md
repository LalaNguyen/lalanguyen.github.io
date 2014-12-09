# django_test

Small project to play a little bit with **Django** and **Tastypie** with **neo4j** as a db using **neomodel**.

It implements some demo models, and offers interaction with them through a **restful API**

You'll find API documentation on http://localhost:8000/api/v1/doc/ thanks to **tastypie_swagger**


## How to use it

```
$ git clone git@github.com:4d-life/test-neo4j.git django_neo4j_test
$ cd django_neo4j_test/
$ vagrant up --provision
$ vagrant ssh
vagrant@precise64:~$ cd /vagrant/
vagrant@precise64:/vagrant$ source /home/vagrant/4dlife-env/bin/activate
(4dlife-env)vagrant@precise64:/vagrant$ cd src/
(4dlife-env)vagrant@precise64:/vagrant/src$ python manage.py runserver 0.0.0.0:8000    
```

Go to you browser and open **http://localhost:8000/api/v1/?format=json** or **http://localhost:8000/api/v1/doc/**. This is the Django app.

If you go to **http://localhost:7474** you'll find yourself with Neo4j's admin interface.

You can reload the default database by just executing:

```
$ install_demo_db.sh
```

Or if you prefer to start with an empty database to do your own checks:

```
$ dropneo4jdb.sh
```

## Testing query times on the Neo4j database

This was done in order to know if Neo4j suited OK for what 4d-life needs were.

Log in the machine and go to /vagrant/src. Activate the virtual environment ```source /home/vagrant/4dlife-env/bin/activate``` and start a console with ```./manage.py shell_plus```. Execute this code:

```python
from neomodel import db
import time

current_milli_time = lambda: int(round(time.time() * 1000))

queries = ['match (n:Character),(w:World {name: "W5"}) where not (n)-[:FRIEND]-() and (n)-[:LIVES_IN]->(w) return n.name',
           'match (w:World {name: "W5"})-[r2:LIVES_IN]-(n:Character)-[r:FRIEND]-(m) where not (m)-[:LIVES_IN]-(w) return n.name',
           'match (p:Character)-[r:LIVES_IN]-(w:World), (p)-[r2:FRIEND]-(p2:Character)-[r3:LIVES_IN]-(w) return distinct(p.name)',
           'match (n:Character)-[r:ENEMY]-() return n.name, count(n) as num ORDER BY num desc LIMIT 1',
           'match (p:Character)-[r:LIVES_IN]-(w:World) return toFloat(count(r)) / count(distinct w)',
           'match (p:Character)-[r:ENEMY]-(p2:Character)-[r2:FRIEND]-(p3:Character) return p.name,collect(DISTINCT(p3.name)) order by p.name',
           'match (w:World {name: "W5"})-[:LIVES_IN]-(p:Character)-[r:ENEMY]-(p2:Character)-[r2:FRIEND]-(p3:Character) return p.name,collect(DISTINCT(p3.name)) order by p.name',]

result = {}
for query in queries:
    timing = []
    for i in range(0, 3):
        t0 = current_milli_time()
        results, meta = db.cypher_query(query)
        t1 =  current_milli_time()
        timing.append(t1 - t0)
    avg = sum(timing) / float(len(timing))
    timing.append('avg = ' + str(avg))
    result[query] = timing
print result
```

Refer to the iPython documentation in order to know how to do that. Look for %cpaste.

Anyway, the timings given looked right on the default VM created (not too many resources: 512 MB RAM and only 1 processor). Here are the timings on my machine:

{'match (n:Character)-[r:ENEMY]-() return n.name, count(n) as num ORDER BY num desc LIMIT 1': [1079, 231, 235, 'avg = 515.0'], 'match (w:World {name: "W5"})-[r2:LIVES_IN]-(n:Character)-[r:FRIEND]-(m) where not (m)-[:LIVES_IN]-(w) return n.name': [229, 81, 81, 'avg = 130.333333333'], 'match (p:Character)-[r:LIVES_IN]-(w:World), (p)-[r2:FRIEND]-(p2:Character)-[r3:LIVES_IN]-(w) return distinct(p.name)': [2740, 640, 642, 'avg = 1340.66666667'], 'match (p:Character)-[r:ENEMY]-(p2:Character)-[r2:FRIEND]-(p3:Character) return p.name,collect(DISTINCT(p3.name)) order by p.name': [17570, 14026, 13932, 'avg = 15176.0'], 'match (p:Character)-[r:LIVES_IN]-(w:World) return toFloat(count(r)) / count(distinct w)': [87, 39, 22, 'avg = 49.3333333333'], 'match (n:Character),(w:World {name: "W5"}) where not (n)-[:FRIEND]-() and (n)-[:LIVES_IN]->(w) return n.name': [2885, 425, 203, 'avg = 1171.0'], 'match (w:World {name: "W5"})-[:LIVES_IN]-(p:Character)-[r:ENEMY]-(p2:Character)-[r2:FRIEND]-(p3:Character) return p.name,collect(DISTINCT(p3.name)) order by p.name': [259, 144, 152, 'avg = 185.0']}

The important data here is the 'avg'.

