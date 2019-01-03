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

__Getting a station code__

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

__Downloading the weather data__

Now that we have the code, we can use it do download a spreadsheet of weather data from the Environment Canada API. The documentation explains that we can get the data from the following URL:

http://climate.weather.gc.ca/climate_data/bulk_data_e.html?format=csv&stationID=${ID}&Year=${year}&Month=${month}&Day=${day}4&timeframe=${tf}&submit=Download+Data

The station ID, year, month, and day are specified as shown, along with a timeframe, 1=hourly, 2=daily, 3=monthly. 

Making the substitutions for Trudeau Airport (ID=30165) gives us:

```
$ URL="http://climate.weather.gc.ca/climate_data/bulk_data_e.html?format=csv&stationID=30165&Year=2018&Month=1&Day=1&timeframe=2&submit=Download+Data" 
$ curl -s $URL | head
"Station Name","MONTREAL/PIERRE ELLIOTT TRUDEAU INTL"
"Province","QUEBEC"
"Current Station Operator","Environment and Climate Change Canada - Meteorological Service of Canada"
"Latitude","45.47"
"Longitude","-73.74"
"Elevation","32.10"
"Climate Identifier","702S006"
"WMO Identifier","71183"
"TC Identifier","WTQ"
```

This is a bunch of indentifying information about the station. We also set a variable for the URL so we don't have to keep looking at it. Eventually it contains rows of data:

```
$ curl -s $URL | tail
"2018-12-22","2018","12","22","","8.5","","-6.1","","1.2","","16.8","","0.0","","","","","M","2.0","","1","","28","","56",""
"2018-12-23","2018","12","23","","-6.1","","-11.0","","-8.5","","26.5","","0.0","","","","","M","0.2","","1","","25","","39",""
"2018-12-24","2018","12","24","","-6.4","","-11.3","","-8.8","","26.8","","0.0","","","","","M","0.0","","1","","","","",""
"2018-12-25","2018","12","25","","-8.2","","-13.5","","-10.8","","28.8","","0.0","","","","","M","0.0","","1","","","","",""
"2018-12-26","2018","12","26","","-4.3","","-12.1","","-8.2","","26.2","","0.0","","","","","M","0.2","","1","","","","",""
"2018-12-27","2018","12","27","","-8.4","","-14.4","","-11.4","","29.4","","0.0","","","","","M","0.6","","2","","5","","31",""
"2018-12-28","2018","12","28","","3.0","","-8.7","","-2.9","","20.9","","0.0","","","","","M","14.6","","2","","16","","32",""
"2018-12-29","2018","12","29","","6.4","","-12.2","","-2.9","","20.9","","0.0","","","","","M","0.6","","2","","25","","51",""
"2018-12-30","2018","12","30","","-6.9","","-12.7","","-9.8","","27.8","","0.0","","","","","M","0.5","","2","","","","",""
"2018-12-31","2018","12","31","","2.2","","-8.4","","-3.1","","21.1","","0.0","","","","","M","2.5","","2","","15","","38",""
```

This is the end of 2018. We will have to play around a bit to figure out where the columns headers are in the file:

``` 
$ curl -s $URL | head -30 | tail -5
"Date/Time","Year","Month","Day","Data Quality","Max Temp (°C)","Max Temp Flag","Min Temp (°C)","Min Temp Flag","Mean Temp (°C)","Mean Temp Flag","Heat Deg Days (°C)","Heat Deg Days Flag","Cool Deg Days (°C)","Cool Deg Days Flag","Total Rain (mm)","Total Rain Flag","Total Snow (cm)","Total Snow Flag","Total Precip (mm)","Total Precip Flag","Snow on Grnd (cm)","Snow on Grnd Flag","Dir of Max Gust (10s deg)","Dir of Max Gust Flag","Spd of Max Gust (km/h)","Spd of Max Gust Flag"
"2018-01-01","2018","01","01","","-18.8","","-25.3","","-22.1","","40.1","","0.0","","","M","","M","0.0","","18","","25","","32",""
"2018-01-02","2018","01","02","","-14.0","","-24.9","","-19.5","","37.5","","0.0","","","M","","M","2.5","","18","","13","","32",""
"2018-01-03","2018","01","03","","-10.2","","-16.2","","-13.2","","31.2","","0.0","","","M","","M","0.2","","22","","","","<31",""
"2018-01-04","2018","01","04","","-6.7","","-14.1","","-10.4","","28.4","","0.0","","","M","","M","0.9","","23","","27","","46",""
```

A lucky guess. The 26th row is contains the headers.

```
$ curl -s $URL | sed 's/"//g' | awk -F, 'NR==26 {for (i=1;i<=NF;i++) {print i, $i}}'
1 Date/Time
2 Year
3 Month
4 Day
5 Data Quality
6 Max Temp (°C)
7 Max Temp Flag
8 Min Temp (°C)
9 Min Temp Flag
10 Mean Temp (°C)
11 Mean Temp Flag
12 Heat Deg Days (°C)
13 Heat Deg Days Flag
14 Cool Deg Days (°C)
15 Cool Deg Days Flag
16 Total Rain (mm)
17 Total Rain Flag
18 Total Snow (cm)
19 Total Snow Flag
20 Total Precip (mm)
21 Total Precip Flag
22 Snow on Grnd (cm)
23 Snow on Grnd Flag
24 Dir of Max Gust (10s deg)
25 Dir of Max Gust Flag
26 Spd of Max Gust (km/h)
27 Spd of Max Gust Flag
```

We followed the same pattern as with the station names file to display the fields.

Here are a few of the mean temperatures (column 10) by date:

```
$ curl -s $URL | sed 's/"//g' | sed -n '27,36p' | awk -F, '{print $1, $10}'
2018-01-01 -22.1
2018-01-02 -19.5
2018-01-03 -13.2
2018-01-04 -10.4
2018-01-05 -18.3
2018-01-06 -21.9
2018-01-07 -17.7
2018-01-08 -7.1
2018-01-09 -5.3
2018-01-10 -8.0
```

Having read in some data, we want to do some calculations on it. For example, get the average monthly temperature. Consider:

```
$ curl -s $URL | sed -e 's/"//g' | awk -F, 'NR>26 {sum[$3]+=$10; num[$3]+=1} END {for (k in sum) print k,sum[k]/num[k]}' | sort -n
01 -9.71613
02 -4.56429
03 -0.919355
04 3.92
05 14.8161
06 18.3167
07 24.2129
08 22.2645
09 17.63
10 6.05161
11 -0.716667
12 -4.81613
```

This statement uses two new features of awk. One is array indexing. The statement `sum[$3]+=$10` uses the month (field 3) as an index to an array called `sum`. Indices previously not encountered are initialized to 0. The mean temperature that day (field 10) is then added to the sum. At the same time, we use `num[$3]+=1` to count the number of days summed for each month.

The second feature is the END statement for awk. This is what is executed after we have processed all lines in the file. In this case, we are printing out the averages for each month, obtained by dividing the sums by the counts.

The resulting experession is piped to sort because awk does not necessarily step though the array indices in order - they are strings, not numbers.

As an aside, awk's array indexing works with any strings:

```
$ curl -s http://www.gutenberg.org/files/108/108-0.txt | tr '[:upper:]' '[:lower:]' | tr -cd '[a-z]\n ' | tr -s '\n' | tr ' ' '\n' | awk '{words[$1]+=1} END { for (w in words) print w, words[w]}' | sort -k 2 -r -n | head
the 6430
and 2955
of 2927
i 2910
a 2721
to 2682
that 2107
in 1900
was 1816
it 1814
```

The line above downloads the text of a book, uses tr to change all letters to lowercase and remove non-letters, puts one work on each line (by replacing spaces with newlines) and then uses awk to count the occurrence of each work. The statement 'words[$1]+=1` is all it takes to use the word on the current line as an index into the array and add one to the count for that word.

Lastly, we can do more complicated things if we want. It may be better off to use python at this point, but for big files or remote access the following idea may still make sense. Here we calculate the standard deviation for each month (I adapted this from another tutorial available at [http://john-hawkins.blogspot.com/2013/09/using-awk-for-data-science.html]

First save the data in a file and get some of the preprocessing out of the way:

```
$ curl -s $URL | sed -e 's/"' | awk 'NR>26' > TMPFILE
```

Now, compute the standard deviation in two passes:

```
$ awk -F, 'pass==1 {sum[$3]+=$10; num[$3]+=1} pass==2 { mean=sum[$3]/num[$3]; ssd[$3]+=($10-mean)*($10-mean)} END {for (k in sum) print k,sum[k]/num[k], sqrt(ssd[k]/num[k])}' pass=1 TMPFILE pass=2 TMPFILE | sort
01 -9.71613 7.37997
02 -4.56429 5.49586
03 -0.919355 4.14825
04 3.92 4.79794
05 14.8161 5.24731
06 18.3167 4.86991
07 24.2129 2.57716
08 22.2645 4.70857
09 17.63 4.76873
10 6.05161 4.88473
11 -0.716667 5.41738
12 -4.81613 4.82006
```

You can see we pass the data to awk twice along with a `pass` variable. On the first pass, we find the mean for each month as before. On the second pass, we add the squared residual value for each day to an array for its month ( `ssd[$3]+=($10-mean)*($10-mean)` ) and then take the square root of the average to get the standard deviation.

This gives an idea of how a more complex analysis could take place. For example the link referenced above shows an example of using awk to calculate the correlation between two variables. 

While awk does most of the heavy lifting, combining it with sed, grep, and a few other utilities creates a simple and powerful workflow that can handle many simple data manipulation and analysis tasks.


