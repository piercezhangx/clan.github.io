---
layout: default
title: demo of rrdtool
---
rrdtool 的演示代码

<pre>
#!/bin/sh

rm -fr count.rrd gauge.rrd

rrdtool create rrdtool.rrd --step 60 DS:gauge:GAUGE:120:0:U DS:count:COUNTER:120:0:U RRA:AVERAGE:0.5:1:60

start=$(date --date='20121204' +%s)

for i in `seq 1 60`; do
    t=$((start + i * 60))
    rrdtool update rrdtool.rrd --template gauge:count ${t}:${i}:$((i * i * 2))
done

WIDTH=504
HEIGHT=128
DATETIME=$(date)
INTERVAL=$((24 * 3600))

rrdtool graph rrdtool.png -i -M -s ${start} -e $((start + 3600)) \
    -w ${WIDTH} -h ${HEIGHT} \
    -l 0 \
    DEF:rcount=rrdtool.rrd:count:AVERAGE \
    DEF:rgauge=rrdtool.rrd:gauge:AVERAGE \
    CDEF:count=rcount,1,* \
    CDEF:gauge=rgauge,10,/ \
    LINE1:count#00ff00:"count" \
    LINE1:gauge#0000ff:"gauge"
</pre>
{{ page.date | date_to_string }}
