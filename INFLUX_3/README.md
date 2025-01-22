### Домашняя работа 3

- есть два набора данных по ценам на фрукты и овощи<br> 
- в Европейских странах за условные 150 дней<br>
- фрукты - **fruits_dataset.csv**<br>
- овощи  - **veggies_dataset.csv**<br>
- колличество стран и записей в файлах одинаково по 7 фруктов и овощей по 150 дней
- загрузим данные с помощью программы на Python
```python3
import datetime

first_date = datetime.datetime(year=2024, month=6, day=1, hour=6)
delta = datetime.timedelta(days=1)

count = 0

import influxdb_client 
from influxdb_client.client.write_api import SYNCHRONOUS

URL = "http://192.168.1.244:8086"
BUCKET = "test_3"
ORG = "sirus"
TOKEN = "ICEmDRlB94HhXZmur02hlxEPTe1QCIOE7_j0V-ivdwAokYGs132qpIpDqKkGQtuTZ4vgwnEP3lJe-8EEfaS6lw=="

client = influxdb_client.InfluxDBClient( url=URL, token=TOKEN, org=ORG )
write_api = client.write_api( SYNCHRONOUS )

with open('fruits_dataset.csv','r') as file01:
    with open('veggies_dataset.csv','r') as file02:
        with open('result_dataset.line','w') as file03:
            line = file01.readline()
            line = file02.readline()
            while True:
                line = file01.readline()
                if not line:
                    break
                block1 = tuple(line.strip().replace('"','').split(','))
                line = file02.readline()
                block2 = tuple(line.strip().replace('"','').split(','))
                if (count % 150) == 0:
                    next_date = first_date
                count = count + 1    
                dcount = count // 150
                xtime = int(next_date.timestamp()*1000000000)
                next_date = next_date + delta
                #pline1 = 'test' + ',country=' + block1[0] + ',fruit=' + block1[1] + ' price=' + block1[3] + ' ' + str(xtime)    
                #file03.write(pline1+"\n")
                p1 = influxdb_client.Point('Europe').time(xtime).tag("country",block1[0]).tag("fruit",block1[1]).field("price",float(block1[3]))
                write_api.write( BUCKET, ORG, p1 )
                file03.write( p1.to_line_protocol()+"\n" )
                #pline2 = 'test' + ',country=' + block2[0] + ',veggie=' + block2[1] + ' price=' + block2[3] + ' ' + str(xtime)    
                #file03.write(pline2+"\n")
                p2 = influxdb_client.Point('Europe').time(xtime).tag("country",block2[0]).tag("veggie",block2[1]).field("price",float(block2[3]))
                write_api.write( BUCKET, ORG, p2 )
                file03.write( p2.to_line_protocol()+"\n" )
                pass
```

