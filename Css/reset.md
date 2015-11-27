# CSS 通用规范
##代码组织

##1模块组织


###1.1组件Components
从 Components 的角度思考，将网站的模块都作为一个独立的 Components。

**组件命名Naming components**

最少以两个单词命名，通过 - 分离，例如：
```
点赞按钮 (.like-button)
搜索框 (.search-form)
文章卡片 (.article-card)
```

###1.2 元素Elements 
Elements 是 Components 中的元素

**元素命名Naming elements**
```
//Elements 的类名应尽可能仅有一个单词。

 .search-form {
    .field { /* ... */ }
    .action { /* ... */ }
  }

//On multiple words （多个单词）
对于倘若需要两个或以上单词表达的 Elements 类名，不应使用中划线和下划线连接，应直接连接。

  .profile-box {
    .firstname { /* ... */ }
    .lastname { /* ... */ }
    .avatar { /* ... */ }
  }

```



### 1.3Layout （布局）
* Components 应该在不同的上下文中都可以复用，所以应避免设置以下属性：
```
Positioning (position, top, left, right, bottom)
Floats (float, clear)
Margins (margin)
Dimensions (width, height) *
Fixed dimensions （固定尺寸）
```

* 头像和 logos 这些元素应该设置固定尺寸（宽度，高度...）。

* 在父元素中设置定位
倘若你需要为组件设置定位，应将在组件的父元素中进行处理，比如以下例子中，将 widths 和 floats 应用在 list component(.article-list) 当中，而不是 component(.article-card) 自身。
```
  .article-list {
    & {
      @include clearfix;
    }

    > .article-card {
      width: 33.3%;
      float: left;
    }
  }

  .article-card {
    & { /* ... */ }
    > .image { /* ... */ }
    > .title { /* ... */ }
    > .category { /* ... */ }
  }
```

* 避免过多嵌套
当出现多个嵌套的时候容易失去控制，应保持不超过一个嵌套。
```
  /* ✗ Avoid: 3 levels of nesting */
  .image-frame {
    > .description {
      /* ... */

      > .icon {
        /* ... */
      }
    }
  }

  /* ✓ Better: 2 levels */
  .image-frame {
    > .description { /* ... */ }
    > .description > .icon { /* ... */ }
  }
```

###1.4 总结

以 Components 的角度思考，以两个单词命名（.screenshot-image）
Components 中的 Elements，以一个单词命名（.blog-post .title）
Variants，以中划线-作为前缀（.shop-banner.-with-icon）
Components 可以互相嵌套
记住，你可以通过继承让事情变得更简单



##2. 属性的排列顺序
_如：以下2种顺序均可_
相关属性应为一组，推荐的样式编写顺序

Positioning
Box model
Typographic
Visual
由于定位（positioning）可以从正常的文档流中移除元素，并且还能覆盖盒模型（box model）相关的样式，因此排在首位。盒模型决定了组件的尺寸和位置，因此排在第二位。

其他属性只是影响组件的内部（inside）或者是不影响前两组属性，因此排在后面。

.declaration-order {
  /* Positioning */
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 100;

  /* Box model */
  display: block;
  box-sizing: border-box;
  width: 100px;
  height: 100px;
  padding: 10px;
  border: 1px solid #e5e5e5;
  border-radius: 3px;
  margin: 10px;
  float: right;
  overflow: hidden;

  /* Typographic */
  font: normal 13px "Helvetica Neue", sans-serif;
  line-height: 1.5;
  text-align: center;

  /* Visual */
  background-color: #f5f5f5;
  color: #fff;
  opacity: .8;

  /* Other */
  cursor: pointer;
}

