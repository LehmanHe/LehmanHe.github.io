---
title: ListView复用机制
date: 2016-05-23 20:12:06
tags: Android
---

convertView什么时候是NULL？
今天在stackoverflow上看到大神回答的，翻译过来加深印象。第一次翻译，错了别光打脸。
[我是传送门](https://stackoverflow.com/questions/11945563/how-listviews-recycling-mechanism-works/14108676#14108676?newreg=774a8684c47b4c128feb82c4f70b24a9)
刚开始的我也没有搞懂listview中的复用机制，但是经过几天的琢磨看到了下面的图片后终于有了眉目。图片来自android.amberfog 
![这里写图片描述](http://i.stack.imgur.com/VLG9g.jpg)

当你的listview适配了一个含很多数据的adapter，并且显示在屏幕上的时候，即使在你上下滑动listview的时候，它的行数并没有增加，这正是listview快速有效的原因所在。如上图所示，刚开始有7个可见的item，当你向上滑动listview时，item1变得不可见了，此时item1就被放入了recycler中，你可以在代码中加入下列代码来证实：

```
System.out.println("getview:"+position+" "+convertView);
```
假如你的getview()代码是：

```
public View getView(final int position, View convertView, ViewGroup parent)
{
    System.out.println("getview:"+position+" "+convertView);
    View row=convertView;
    if(row==null)
    {
        LayoutInflater inflater=((Activity)context).getLayoutInflater();
        row=inflater.inflate(layoutResourceId, parent,false);

        holder=new PakistaniDrama();
        holder.tvDramaName=(TextView)row.findViewById(R.id.dramaName);
        holder.cbCheck=(CheckBox)row.findViewById(R.id.checkBox);

        row.setTag(holder);

    }
    else
    {
        holder=(PakistaniDrama)row.getTag();
    }
            holder.tvDramaName.setText(dramaList.get(position).getDramaName());
    holder.cbCheck.setChecked(checks.get(position));
            return row;
    }
}
```
观察你的logcat你会发现，刚开始显示那7个item的时候convertview都是NULL，这是因为刚开始的时侯Recycler 中没有可以复用的view（item），所以getview（）就为每一个item创建一个新的view。但是当你把item1向上划到屏幕外不可见时，item1就被放入了Recycler 。之后你再滑动产生新的item时就不再重新创建view了，而是复用Recycler 中的view（item）。如果你的position0的地方（即item1中放置了一个checkbox，并且已经选中了，你会发现item8中的checkbox也是选中的）。

另外：
1.永远不要把listview的layout_height 和layout_width设置为wrap_content 。因为getview（）会强制adapter去获取子view的高度（用作listview显示的参数），而且有时候会产生一些意想不到的结果，例如你没有滑动listview却返回了convertview。所以建议使用match_parent或固定值。
