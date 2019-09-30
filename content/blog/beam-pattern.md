---
title: "Beam Pattern: Calling External Services Using Retrofit For Data Enrichment"
date: 2019-07-22T08:32:41+07:00
draft: false
image: "/images/blog/apache-beam.jpg"
tags: ["beam", "kotlin", "english", "data engineering"]
types: "blog"
comments: true
---

## Background

For the past two months, my team and I had deployed pipeline using Apache Beam in a production environment. The pipeline step is very simple just filter data by event, enrich data by user id, then insert to BigQuery.
We expect that every event that will flow to the pipeline will be stored in BigQuery immediately (real-time).
As you can guess, what we had in mind are definitely different from what happens in reality. Watermark is delayed up to 20 hours, isn't it insane?

## Attempt

The first suspect is pubsub attributes(timestamp attribute) because we only use attribute id. As watermark is a sign of completeness, so we think if we add timestamp attribute to pubsub then the watermark will be advanced and not delayed like that. Is it a success? well, the answer is no.

The second suspect is fixed windows after reading pubsub. As we just use simple windows, we think that we need to customize trigger in fixed windows. After the customization of trigger and watermark as mentioned in beam documentation, the pipeline is still delayed. After reading beam documentation and try to debug the pipeline using DirectRunner, we found that we can remove fixed windows. The reason is that when pubsub message flow to the pipeline, it can use global windows created in the pipeline.
