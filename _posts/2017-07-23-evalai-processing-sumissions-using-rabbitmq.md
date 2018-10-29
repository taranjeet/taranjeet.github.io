---
layout: post
title: EvalAI&#58; Processing submissions using RabbitMQ
date: 2017-07-23 13:30
tag:
- opensource
- gsoc
- rabbitmq
category: blog
author: taranjeet
description: This post is about rabbitmq is used to reduce submission time in evalai, which is a platform for ai researchers and data scientists.
---

[EvalAI](https://github.com/cloud-cv/evalai) is a platform which helps AI researchers, students and data scientists to collaborate and participate in various AI challenges. The platform hosts various challenges, corresponding to which participants make their submission.

## Submission

A submission is a prediction made against a Challenge Phase. It is processed using the evaluation script of a challenge, sometimes involving heavy computations.

## Processing Submission

A submission can be processed either asynchronously or synchronously.

Processing a submission synchronously will __defer__ the response for the request until the processing is not complete. The processing involves __heavy computation__, so it may be a bottleneck when the number of concurrent submissions is made.

This bottleneck can be easily handled if the submission messages are processed in an asynchronous way. All the submission made can be __queued__, which can be processed by workers in a round-robin fashion. This can be easily horizontally scaled by adding more workers if the number of submission messages increases.

## RabbitMQ: The Saviour

To process messages in an asynchronous way, we at [EvalAI](https://github.com/cloud-cv/evalai) are using RabbitMQ, a Queue-based system.

Whenever a submission is made, the response is sent immediately after saving a submission instance and a message is queued in an asynchronous way. This message is then picked up by the worker and processed further.

Situations wherein the number of submission increases and the workers are already processing at their maximum capacity can be handled easily as every message is eventually queued up. The message persists in the queue until it is not picked up and acknowledged by the worker. This way, we can easily handle and process a large number of submission even at peak times.

For processing messages, a python worker listens on the queue. The worker picks up the message from the queue, processes the submission and saves its status in the database.

This way, we are using [RabbitMQ](https://www.rabbitmq.com) at [EvalAI](https://github.com/cloud-cv/evalai) to process submissions made against a challenge.
