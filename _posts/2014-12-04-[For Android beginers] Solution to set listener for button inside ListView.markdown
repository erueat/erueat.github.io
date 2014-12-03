---
layout: post
title:  "[For Android beginers] Solution to set listener for button inside ListView"
date:   2014-12-04 01:52:31
categories: blogs
---
**ListView** is often used when we develop Android applications. Generally, we need to add buttons inside each Item. When it comes to add `OnClickListener` for these buttons, we may want to `new` one `OnClickeListener` for each Button inside the `getView()` method of the adapter of the ListView. However, this solution generates lots of listeners and will behave badly on performance, while these listerners behave the same as each other, except for the caller's id. So, how can we optimize the solution for better performance?

Now that we know every listener behave the same, we can firstly generate listener by Singleton Mode. For Example:

{% highlight java %}
class MyOnClickListener implements OnClickListener {

    private static MyOnClickListener instance = null;
       
    private MyOnClickListener() {
    }

    public static MyOnClickListener getInstance() {
        if (instance == null) 
            instance = new MyOnClickListener() ;
        return instance;
    }

    @Override
    public void onClick(View view) {
        //TODO: do something here
    }
}
{% endhighlight %}

And in `getView`, after we've got the instance of the button, we just need to call `button.setOnClickListener(MyOnClickListner.getInstance());` to set listener for it. In this way, we ensure that each button shares the same listener.

However, we often need more. We may need to know which button was clicked, then we can behave differently according to the index of the button. 

In non-singleton mode, we may choose to construct different listeners with different indices. But this would not work any more because we are using singleton mode. So there is only one choice, that is to make the button itself know who it is. Now the solution becomes clear: we need to override the `Button` class to give an `index` info to it. Codes may be like:

{% highlight java %}
class MyButton extends Button {

    private int index = -1;

    public int getIndex() {
        return index;
    }

    public void setIndex(int index) {
        this.index = index;
    }

    public MyButton(Context context) {
        super(context);
        // TODO: do something here if you want
    }

    public MyButton(Context context, AttributeSet attrs) {
        super(context, attrs);
        // TODO: do something here if you want
    }

    public MyButton(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        // TODO: do something here if you want
    }
}
{% endhighlight %}

Then we need to change `<Button></Button>` into `<your.package.name.MyButton></your.package.name.MyButton>` inside the xml file of the ListView item. We convert the return value of `findViewById` in `getView` to `MyButton`, invoke `setIndex` and pass the position of the item to the function. As for `MyOnClickListener`, we convert the parametre of `onClick` to MyButton, and get the index info by calling `getIndex`.

Inside the `adapter`:
{% highlight java %}
// ....
MyButton button = null;
// ....
@Override
public View getView(int position, View convertView, ViewGroup parentView) {
    View view = convertView;
    if (convertView == null) {
        view = LayoutInflater.from(activity).inflate(R.layout.YOUR_ITEM_LAYOUT, null);
    }

    // ....

    button = (MyButton) view.findViewById(R.id.YOUR_BUTTON_ID);
    button.setIndex(position);
    button.setOnClickListener(MyOnClickListener.getInstance());
}
{% endhighlight %}

And inside `MyOnClickListener`:
{% highlight java %}
// ....

@Override
public void onClick(View view) {

    int index = ((MyButton)view).getIndex();

    // ....
}
{% endhighlight %}

Now we've achieved the initial purpose: using the same listener for buttons in different items of a ListView. What's more. this solution makes it easy to share other data between button and listener: no matter what you need to share, you just need to add a pair of getter and setter for the data in MyButton. And if your listener need to know more about the adapter, just add arguments to the constructor of listener. 