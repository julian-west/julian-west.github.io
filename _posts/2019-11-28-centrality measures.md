---
title: "Custom Centrality Measure - Suicide Risk Identification"
categories:
  - Blog
tags:
  - Python
  - Networkx
  - Network Analytics
description: "Suicide pacts between friends is a big concern in sociology of young adults. Can network analytics help identify at risk pairs of students?"
excerpt: "Custom centrality metrics for suicide risk identification"
header:
  teaser: /assets/centrality_measures_files/network.png
image: /assets/centrality_measures_files/network.png
---

<img src='{{site.baseurl}}/assets/centrality_measures_files/network.png' style="width:400px;height:400px;">
<br>

**Jupyter notebook available at [this](https://github.com/julian-west/network-centrality-suicide-prevention) GitHub repository**

**To view the notebook in nbviewer [click here](https://nbviewer.jupyter.org/github/julian-west/network-centrality-suicide-prevention/blob/master/Network%20centrality%20measures%20-%20suicide%20risk.ipynb)**

Suicide pacts between students is a concern in sociology of young adults. Analysis of student networks can yield important information about relationships between students and identify which students have close relationships and could be in a suicide pact with each other. Should an 'important' member of the network become a suicide risk, it might be possible to proactively reach out to other 'close by' members of the network to help them.

This analysis project uses the phone call records of 27 students to extract information about the most influential nodes in the network and also assess behavioural information such as how many calls they make or how many different people they are interacting with. These insights will be used to develop a method of predicting which students in the network might be involved in a suicide pact given that another member in the network has already been identified as at risk. This information can be used by the student support team to proactively reach out to at risk students.

### Brief
> A high-school Suicide and Distress Counseling service has data on the phone calls between 27 high-school students (a who-talks-to-whom network). One of its directors believes network analytics will help it proactively monitor and help students in distress.<br><br>
One of the concerns in sociology of young adults is the prevalence of suicide-pacts between pairs of friends. The Director wants to come up with a measure so that if they identify a person as potentially troubled, they can identify a “paired” subject that they can reach out pro-actively to assess suicide risk. It is known that potential at-risk students become increasingly withdrawn<br><br>
Given this information, develop a centrality measure which identifies the most at risk students in the network.

### Executive Summary
The phone call records of 27 students at a school were analysed to identify important connections between students and between mutual third parties in the network. A centrality measure was proposed to identify which connected parties were most at risk of suicide given that a student already in the network had been identified as at risk of suicide. i.e. which nodes have influence over over nodes in the network. The centrality measure used two terms, the strength of relationship between a students and the 'reclusiveness' of an individual, to calculate a risk score. Students who had a strong relationship with the target node and exhibited signs of reclusiveness were deemed as higher risk. No 'ground truth' is known for this network so the effectiveness of the centrality measure was tested by applying it to the text message records of the students to assess if the same outcome was reached. The same students were indeed identified as at risk on both the phone call records dataset and the text messages dataset.


Please see the GitHub repository for the full analysis which covers, exploratory data analysis, creating the custom centrality measure and developing a method for identifying at risk students.

