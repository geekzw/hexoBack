---
title: java8集合针对对象属性排序
date: 2018-05-01 23:36:28
tags:
---
集合对按照属性去重，方法不止一种，如果对象是自己的，重写了equals和hashCode的对象就更容易一些。当对象没有重写这两个方法，又不能去重写的时候，实现就有些不雅观了。java8出现后，集合的操作变的简单又优雅，所以想用java8的stream实现。最后找到了这个方式
<!-- more-->
``java

     static class Persion{
        private String id;

         public Persion(String id) {
             this.id = id;
         }

         public String getId() {
            return id;
        }

        public void setId(String id) {
            this.id = id;
        }

         @Override
         public String toString() {
             return id;
         }
         
     }
     
     public static void main(String[] args){
        List<Persion> strs = new ArrayList<>();
        strs.add(new Persion("1"));
        strs.add(new Persion("2"));
        strs.add(new Persion("3"));
        strs.add(new Persion("1"));
        List<Persion> unique = strs.stream().collect(
                Collectors.collectingAndThen(
                        Collectors.toCollection(() -> new TreeSet<>(Comparator.comparing(Persion::getId))), ArrayList::new)
        );
    	System.out.print(Arrays.toString(unique.toArray()));
    }
``


原理待补充。。。。