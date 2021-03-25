---
title: 图形学笔记
date: 2021-03-19 23:30:12
tags:
---

# 光线追踪
ray = o + td

## 平面求交
(p - p‘) * N = 0

## AABB
axis aligned bounding box

xx - intr with ray, t…min x, t…max x;
yy - intr with ray, t…min y, t…max y;
zz - intr with ray, t…min z, t…max z;

## acc
uniform grid（teapot in stadium）

oct-tree
kd-tree
//bsp-tree，Binary space partitioning

bvh
