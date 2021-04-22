title: ICMP
author: Jiahao Wu
tags: []
categories:
  - 计算机网络
date: 2021-03-19 23:39:00
---
# ICMP

## ICMP定义和功能

ICMP是基于IP协议工作的，但是它并不是传输层的功能，因此仍然把它归结为网络层协议。它的功能是：  
1. 确认IP包是否成功到达目标地址；  
2. 通知发送过程中IP包被丢弃的原因。  

## ICMP报文格式

0-7bytes：类型；  
8-15bytes：代码；  
15-31bytes：校验和。  
剩下是内容。  
不同类型的报文，内容不同。  
有如下不同类型及含义：  
<table>
  <th>类型</th>
  <th>内容</th>
  <tr>
  <td>0</td>
  <td>回送应答</td>
  <tr>
  <td>3</td>
  <td>目标不可达</td>
  <tr>
  <td>4</td>
  <td>原点抑制</td>
  <tr>
  <td>5</td>
  <td>重定向</td>
  <tr>
  <td>8</td>
  <td>回送请求</td>
  <tr>
  <td>9</td>
  <td>路由器公告</td>
  <tr>
  <td>10</td>
  <td>路由器请求</td>
  <tr>
  <td>11</td>
  <td>超时</td>
  <tr>
  <td>17</td>
  <td>地址子网请求</td>
  <tr>
  <td>18</td>
  <td>地址子网应答</td>
</table>

