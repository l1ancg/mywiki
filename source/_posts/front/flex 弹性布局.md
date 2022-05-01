---
title: flex 弹性布局
date: 2021-10-13 00:18:46
tags:
  - css
  - front
---


开启弹性布局 
display: flex; 
display: inline-flex; 
display: -webkit-flex; /* Safari */

> 盒状模型：依赖 `display` 属性 + `position` 属性 + `float`属性
>
> 弹性布局：flex 对象问容器，容器内子对象为项目

## 容器属性

| 属性              | 说明        | 可选项                                                                          |
| --------------- | --------- | ---------------------------------------------------------------------------- |
| flex-direction  | 主轴方向      | row | row-reverse | column | column-reverse                               |
| flex-wrap       | 如何换行      | nowrap | wrap | wrap-reverse                                               |
| flex-flow       | 前两个合并     | <flex-direction> || <flex-wrap>                                            |
| justify-content | 主轴对齐方式    | flex-start | flex-end | center | space-between | space-around            |
| align-items     | 交叉轴对齐方式   | flex-start | flex-end | center | baseline | stretch                      |
| align-content   | 多根交叉轴对齐方式 | flex-start | flex-end | center | space-between | space-around | stretch |

## 项目属性

| 属性          | 说明                                      | 可选项                |
| ----------- | --------------------------------------- | ------------------ |
| order       | 排列顺序，默认 0                               | 数字                 |
| flex-grow   | 放大比例，默认 0，不放大（如果剩余空间，优先放大 1，全都是 1，则等分）  | 数字                 |
| flex-shrink | 缩小比例，默认 1，不缩小（如果空间不够，优先缩小 0，全都是 0，则等分）  | 数字                 |
| flex-basis  | 分配多余空间前（放大前），设定此项目的主轴占用空间               | 长度值 | auto        |
| flex        | 上面三个合并 flex-grow flex-shrink flex-basis | none | 前两值 | 三个值 |
| align-self  | 单独指定此项目的对齐方式                            | 参考 align-items

