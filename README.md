# TA-detectiontechniquedeepdive

This app provides the case studies and other artefacts shown in the Splunk .conf 2018 presentation "Detection Technique Deep Dive": https://conf.splunk.com/learn/session-catalog.html?search=SEC1039#/

# Case Studies

Each of the case studies can be used by piping to their macro. The macros dynamically generate events that are unique each time the search is run, but exhibit the same properties for the purposes of our analysis. In this way, we can develop detection techniques and run them multiple times to check their efficacy. 

Keep in mind that use of streamstats is always recommended over eventstats for performance. However, there will be situations where you cannot use tstats or mstats as the generating command, precluding the use of streamstats. The slides provide the eventstats form where streamstats isn't strictly necessary, because it will work in more situations, such as here in our small and non-temporal example case studies.

## First Case Study

The first case study is quite straightforward, so I'll let you figure out use of numerical technique (1):
```
| `firstcasestudy`
```
Hint: The first case study has very low variability so only a threshold of 10% above average, i.e. 1.1 is necessary.

## Second Case Study

The second case study is also quite straightforward using numerical techinque (2):
```
| `secondcasestudy`
```

At this point, you may also like to try numerical technique (3) and (4) with the first and second case studies.

As noted in the Methodology on slide 13, we typically want to analyse the behaviour of something such as user, src_ip, etc, so don't forget that you can achieve this by putting those fields in the 'by' list of the numerical techniques' statistical commands (as demonstrated in the Appendix section of the slides).

## Third Case Study

The third case study requires use of the Brown Manoeuvre. Experiment with the difference threshold and you'll see that for this dataset a threshold of 0.3 (i.e. low) should reliably detect the anomaly without false-positives.
```
| `thirdcasestudy`
| `brown_manoeuvre(y)`
| where difference>0.3
```

## Fourth Case Study

The fourth case study has multiple anomalies which are straightforward to find with categorical techniques (A) and (B). However, to find the field2 + field3 tuple which is anomalous by field1, use:
```
| `fourthcasestudy`
| eval tuple=field2.field3
| eventstats count by field1,tuple
| where count==1
```

# Detecting New and First Time Values

If you wish to find "new" or "first time" value/s in temporal data, this is actually the same as finding unique values and so categorical techniques (A) and (B) can be used. The difference in searching for "new" or "first time" events is that we don't just care about their being unique, but also _when_ they occured. For this reason, we must use streamstats instead of eventstats and add a condition that the event occured recently (e.g. in the last hour).

For example, using technique (A) to find when a new value of fieldA suddenly appears:
```
... | streamstats count by fieldA
    | where count==1 AND _time>relative_time(now(),"-1h")
```
As with any alert/correlation search, this would need to be configured to throttle for 1h by fieldA to prevent duplicates, where fieldA would typically be something like user, src_ip, etc.
