---
title: "Basic cache metrics"
date: 2026-01-14
tags:
  - engineering
  - database
  - storage
---

caching metric

## **cache hit ratio**

### **TQ: Total request**

CH: Request hit cache

DBH : Request hit DB

CHR = (CH - DBH) / TQ

## **cache miss ratio**

### **100% - cache hit ratio**

## **cache size**

## **expired rate**

### **Cache hit radio low, Expired rate high: TTL too low**

## **eviction rate**

### **Cache hit radio low, Expired rate low, Eviction rate high: Memory size is too low, need tuning**

Eviction policy

## **FIFO (First in first out)**

### **Init but not reuse in the future**

## **LRU (Least Recently Used)**

### **Hot cold by time, ex: Some data hot when appear and cold down by time**

## **LFU (Least Frequently Used**

### **Hot cold by content, ex: Some data trending and hit frequently when some other not**

Problem

## **Stale Data**

## **Cache invalidation**

### **Write-around**

### **Write DB first, cache async: regular, but when using cache-aside, data might be wrong until cache updated**

### **Write-through**

### **Write DB and cache in the same time: slow but data fresh in both db and cache**

### **Write-behind**

### **Write cache first, db async : quick but data maybe wrong**

### **Solution**

### **Change Data Capture (CDC) thông qua binlog (mysql), oplog (mongodb),**

### **Trigger background job**

### **Setup cronjob để refresh cache**

## **Paging caching**