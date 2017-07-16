title: 从 Formatter 学习 TextWatcher
date: 2015-09-15 21:18:35
tags: 
-  android
categories:
-  android
---

电话号码和银行号码经常会有需要格式话为固定的样式，比如银行卡号通常会格式化为 `XXXX XXXX XXXX XXXX`，手机号通常会格式化为 `XXX XXXX XXXX`，对于手机号码可以用 Android 自带的 `PhoneNumberFormattingTextWatcher`, 但是这种格式化的方式有时候并不是我们想要的，那就需要自己来实现一个 `TextWatcher` 了。

### 第一次尝试实现
先尝试实现一次，在每次text有更新的时候做一次 `format`， 代码如下（省略 `import` 的内容）：

```java
public class FormatterTest extends Activity {

private final char SEPARATOR = ' ';

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.formatter_test);
    EditText editText = (EditText) findViewById(R.id.phone_num);
    editText.addTextChangedListener(new TextWatcher() {

        private boolean mStopFormatting;

        @Override
        public void beforeTextChanged(CharSequence s, int start, int count, int after) {

        }

        @Override
        public void onTextChanged(CharSequence s, int start, int before, int count) {

        }

        @Override
        public void afterTextChanged(Editable s) {
            if(mStopFormatting) {
                return;
            }
            mStopFormatting = true;
            format(s);
            mStopFormatting = false;
        }
    });
}

private String format(String s) {
    SpannableStringBuilder stringBuilder = new SpannableStringBuilder();
    format(stringBuilder);
    return stringBuilder.toString();
}

private void format(Editable s) {
    clean(s);
    if (s.length() < 4) {
        return;
    }
    int i = 0;
    int valid = 0;
    while (i < s.length()) {
        if (valid % 4 == 0 && valid != 0) {
            s.insert(i, Character.toString(SEPARATOR));
            i++;
        }
        i++;
        valid++;
    }
}

private void clean(Editable s) {
    int p = 0;
    while (p < s.length()) {
        if (!Character.isDigit(s.charAt(p))) {
            s.delete(p,p+1);
        }
        p++;
    }
}
}
```

运行之后可以正常的format，不过将光标移动到空格位置删除的时候却无法删除。
原因就是实际上是删除成功了，但是在 `afterTextChanged` 的时候又格式化了，因此空格在删除后又会出现，给人的感觉就是无法删除。

<!--more--> 

### 第二次尝试实现
既然删除了空格又会自动格式化，那我们在格式化之前判断一次，如果删除的这个为 `SEPARATOR`，就停止格式化。
`TextWatcher` 部分代码如下，其他不变。

```java
    editText.addTextChangedListener(new TextWatcher() {

        private boolean mStopFormatting;

        @Override
        public void beforeTextChanged(CharSequence s, int start, int count, int after) {
            if (mStopFormatting) {
                return;
            }
            if (count > 0 && hasSeparator(s, start, count)) {
                mStopFormatting = true;
            }
        }

        @Override
        public void onTextChanged(CharSequence s, int start, int before, int count) {

        }

        @Override
        public void afterTextChanged(Editable s) {
            if (mStopFormatting) {
                mStopFormatting = (s.length() != 0);
                return;
            }
            mStopFormatting = true;
            format(s);
            mStopFormatting = false;
        }
    });
    
private boolean hasSeparator(final CharSequence s, final int start, final int count) {
    for (int i = start; i < start + count; i++) {
        char c = s.charAt(i);
        if (c == SEPARATOR) {
            return true;
        }
    }
    return false;
}
```
这样就解决了空格不能被删除的问题，但是新的问题又来了，当删除一个空格之后，整个 `format` 就失效了。
尽管可以使用，但是用户体验肯定是不好的，改！

### 第三次尝试实现
根据 `API` 的说明，`beforeTextChanged` 和 `onTextChanged` 都不更改 `s` 的内容，我们只能在  `afterTextChanged` 的时候修改字符串。那删除一个空格的时候，我们再加上一个空格，将光标向前移动不就OK了吗？ 代码如下：

```java
            editText.addTextChangedListener(new TextWatcher() {

private boolean mStopFormatting, mDeletedIsSeparator;
private int mDeletedIndex;

@Override
public void beforeTextChanged(CharSequence s, int start, int count, int after) {
    if (mStopFormatting) {
        return;
    }
    if (count > 0 && hasSeparator(s, start, count)) {
        mDeletedIsSeparator = true;
        mDeletedIndex = start;
    }
}

@Override
public void onTextChanged(CharSequence s, int start, int before, int count) {

}

@Override
public void afterTextChanged(Editable s) {
    if (mDeletedIsSeparator) {
        mDeletedIsSeparator = false;
        editText.getEditableText().insert(mDeletedIndex, Character.toString(SEPARATOR));
        editText.setSelection(mDeletedIndex);
        return;
    }
    if (mStopFormatting) {
        return;
    }
    mStopFormatting = true;
    format(s);
    mStopFormatting = false;
}
});
}
```

这次就没有任何问题了。

> 通过这个例子能对 `TextWatcher` 的几个回调函数有比较深刻的理解.