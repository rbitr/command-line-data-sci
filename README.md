# Basic data science at the command line

Linux command line tools can be used to perform many of the key data cleaning and exploration activities that make up data science workflow.

If you ssh into another server, this is a fast way to take a look at data. It also lets you do quick processing without the overhead of writing a full python program. And parsing big files at the command line can speed up subsequent data processing and reduce memory usage by discarding unnecessary data.

The main tools are grep, awk, sed, tr, and a few others, all of which come with ubuntu.

grep is a utility for finding patterns in a file

sed is a stream editor that lets you find and replace text in a file; tr is a simpler utility for finding and replacing text

awk is a programming language that operates on individual lines in a data stream

Linux also has some commands like sort, head, etc.

I will go through an example and explain what the commands and options are doing as the are used:

__Weather data example__

Environment canada has an api that lets you download a .csv file of weather data. The URL for more information is ftp://ftp.tor.ec.gc.ca/Pub/Get_More_Data_Plus_de_donnees/Readme.txt

In order to download the data, you need to specify the code for the weather station. These codes are in a file ftp://client_climate@ftp.tor.ec.gc.ca/Pub/Get_More_Data_Plus_de_donnees/Station%20Inventory%20EN.csv

You can use curl to download this list and save it in a file:
```bash
$ curl -s ftp://client_climate@ftp.tor.ec.gc.ca/Pub/Get_More_Data_Plus_de_donnees/Station%20Inventory%20EN.csv 
```
The file is a csv so we would expect a row of headers and then some data. This one turns out to have some other information at the top as well. Using head to look at the first 5 lines, we get:

```bash
$ < stations.csv head -5
Modified Date: 2018-12-31 23:33 UTC
"Station Inventory Disclaimer: Please note that this inventory list is a snapshot of stations on our website as of the modified date, and may be subject to change without notice."
"Station ID Disclaimer: Station IDs are an internal index numbering system and may be subject to change without notice."
"Name","Province","Climate ID","Station ID","WMO ID","TC ID","Latitude (Decimal Degrees)","Longitude (Decimal Degrees)","Latitude","Longitude","Elevation (m)","First Year","Last Year","HLY First Year","HLY Last Year","DLY First Year","DLY Last Year","MLY First Year","MLY Last Year"
"ACTIVE PASS","BRITISH COLUMBIA","1010066","14","","","48.87","-123.28","485200000","-1231700000","4","1984","1996","","","1984","1996","1984","1996"
```

The fourth line contains the headers and the fifth shows what the first row of data looks like.

Now that we know the headers are on line 4, let's print them out in a way that is easier to see. There are lots of different ways to do this but I like awk. We can start by piping the file into awk and using the NR selector to only show the fourth row:

```bash
< stations.csv awk 'NR==4'
"Name","Province","Climate ID","Station ID","WMO ID","TC ID","Latitude (Decimal Degrees)","Longitude (Decimal Degrees)","Latitude","Longitude","Elevation (m)","First Year","Last Year","HLY First Year","HLY Last Year","DLY First Year","DLY Last Year","MLY First Year","MLY Last Year"
```

Especially in a terminal that wraps, it is nicer to see these as a list. The quickest way is to replace the commas with newlines:

```bash
< stations.csv awk 'NR==4' | tr ',' '\n'
"Name"
"Province"
"Climate ID"
"Station ID"
"WMO ID"
"TC ID"
"Latitude (Decimal Degrees)"
"Longitude (Decimal Degrees)"
"Latitude"
"Longitude"
"Elevation (m)"
"First Year"
"Last Year"
"HLY First Year"
"HLY Last Year"
"DLY First Year"
"DLY Last Year"
"MLY First Year"
"MLY Last Year"
```

An for readibility, let's add row numbers. I will show two ways just for fun. The lazy way, since we already have the command above, is to pipe back into awk and print the row number:

```bash
< stations.csv awk 'NR==4' | tr ',' '\n' | awk '{print NR,$0}'
1 "Name"
2 "Province"
3 "Climate ID"
4 "Station ID"
5 "WMO ID"
6 "TC ID"
7 "Latitude (Decimal Degrees)"
8 "Longitude (Decimal Degrees)"
9 "Latitude"
10 "Longitude"
11 "Elevation (m)"
12 "First Year"
13 "Last Year"
14 "HLY First Year"
15 "HLY Last Year"
16 "DLY First Year"
17 "DLY Last Year"
18 "MLY First Year"
19 "MLY Last Year"
```

awk has an internal variable called NR that is the row number being operated on. And $0 is just the text in the whole line.

The command above is probable the most natural way to do this, because we are discovering what we want to do as we go. If you already knew this was what you wanted to do, you could use awk in one go:

```bash
< stations.csv awk -F, 'NR==4 { for (i=1;i<=NF;i++) { print i, $i}}'
1 "Name"
2 "Province"
3 "Climate ID"
4 "Station ID"
5 "WMO ID"
6 "TC ID"
7 "Latitude (Decimal Degrees)"
8 "Longitude (Decimal Degrees)"
9 "Latitude"
10 "Longitude"
11 "Elevation (m)"
12 "First Year"
13 "Last Year"
14 "HLY First Year"
15 "HLY Last Year"
16 "DLY First Year"
17 "DLY Last Year"
18 "MLY First Year"
19 "MLY Last Year"
```

This way is actually longer, but illustrates a couple things. awk separates the data into columns that can be accessed by $c where c is the column except $0 which gives whole line as in the previous version. The overall syntax of awk is to combine a condition, here NR==4, with what to do if that condition is met. The internal variable NF tells us the number of fields (columns) in the data. Lastly, the switch -F, (or -F ',') tells awk to use a comma as the field separator, because the default is a space.

Now that we know the fields, lets look up stations for a particular city. We can do this easily by using grep to match the city name:

```
$ < stations.csv grep MONTREAL
"MONTREAL LAKE","SASKATCHEWAN","4065260","3390","","","53.62","-105.67","533700000","-1054000000","490.4","1959","1959","","","1959","1959","1959","1959"
"SOUTH MONTREAL LAKE DNR","SASKATCHEWAN","4067670","3396","","","54.05","-105.8","540300000","-1054800000","490.1","1960","1960","","","1960","1960","1960","1960"
"MONTREAL FALLS","ONTARIO","6055300","4084","","","47.25","-84.4","471500000","-842400000","408.4","1932","1955","","","1932","1955","1932","1955"
"MONTREAL FALLS","ONTARIO","6055302","4085","","","47.27","-84.43","471600000","-842600000","306.3","1976","1999","","","1976","1999","1976","1999"
...
```

There is a long list of stations for Montreal. Only the first few are shown above. First, the quotation marks in every line are annoying so lets remove them with sed:

```
$ < stations.csv sed -e 's/"//g' | grep MONTREAL
MONTREAL LAKE,SASKATCHEWAN,4065260,3390,,,53.62,-105.67,533700000,-1054000000,490.4,1959,1959,,,1959,1959,1959,1959
SOUTH MONTREAL LAKE DNR,SASKATCHEWAN,4067670,3396,,,54.05,-105.8,540300000,-1054800000,490.1,1960,1960,,,1960,1960,1960,1960
MONTREAL FALLS,ONTARIO,6055300,4084,,,47.25,-84.4,471500000,-842400000,408.4,1932,1955,,,1932,1955,1932,1955
...
```

sed uses the s command to relace all quotation marks ( /" command ) with nothing ( // ) and do so globally in the file (the g).

What we really care about is the station numnber, and also, if we are looking to find a station that is still in service, the years the station was active. 

```
$ < stations.csv sed -e 's/"//g' | grep MONTREAL | awk -F, '{print $1, $4, $12, $13}'
MONTREAL LAKE 3390 1959 1959
SOUTH MONTREAL LAKE DNR 3396 1960 1960
MONTREAL FALLS 4084 1932 1955
MONTREAL FALLS 4085 1976 1999
MONTREAL RIVER (AUT) 41595 2000 2007
MONTREAL RIVER 4166 1910 1967
MONTREAL ADAC A 8343 1974 1976
MONTREAL ICE CONTROL 5414 1967 1970
MONTREAL/PIERRE ELLIOTT TRUDEAU INTL A 5415 1941 2013
```

Here we show the first, fourth, twelfth and thirteenth columns to get the name, the id, and the operational years. There is still a long list of stations (I showed a few more). So finally let's look at only those still operating in 2018:

```
$ < stations.csv sed -e 's/"//g' | grep MONTREAL | awk -F, '$13>="2018" {print $1, $4, $12, $13}'
MONTREAL INTL A 51157 2013 2018
MONTREAL/ST-HUBERT 48374 2009 2018
MONTREAL/PIERRE ELLIOTT TRUDEAU INTL 30165 2002 2018
MONTREAL MIRABEL INTL A 49608 2012 2018
```

We used the '$13>="2018"' condition with awk in order to only display stations recording in 2018 or later, and now the list is short.




