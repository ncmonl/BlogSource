---
title: RecyclerView排雷--notifyItemRemoved下标（position）不能更新
date: 2017-10-29 23:48:52
categories: "技术栈"
---

## 问题描述##
用以显示列表类型的UI我们经常使用ListView，GridView，Google推出RecyclerView之后，大部分的使用均转换到了RecyclerView中，可定制化十分强，但是使用过程中难免碰到一些问题，以前使用ListView多是使用notifyDataSetChanged（）更新数据，使用RecyclerView删除时，为了使用其自身的删除过渡动画使用notifyItemRemoved删除，那么就碰到了一个问题，再onBindViewHolder中设置了setTag（position）
```Java
@Override
public void onBindViewHolder(RecyclerView.ViewHolder holder,int position){
	...
	holder.itemView.setTag(position)
	...
}
```
直接使用notifyItemRemoved(posiaon)删除会导致该下标不能更新就导致一系列的Bug（/衰）
## 解决方案 ##
查询到了解决办法特此记录一下:为了修复该问题删除之后需要调用notifyItenRangeChanged方法，使下面的itemview重新onBind:
```Java
public void deleteItem(int position){
  mDataList.remove(position);
  notifyItemRemoved(position);
  if(position != mDataList.size()){ // 如果移除的是最后一个，忽略
      notifyItemRangeChanged(position, mDataList.size() - position);
	}
}
```
如果不需要使用动画或更改自定义的动画效果可以添加一下代码
```Java
mRecyclerView.setItemAnimator(newDefaultItemAnimator());
```
