---
title: Cell自适应高度标签
tags: [iOS,Objective-C]
date: 2018-07-08 17:12:24
permalink:
categories: iOS
description: 好久没有更新自己的博客了，去年自己在博客总结说到自己要坚持更新博客，不管是否有技术含量，都要持之以恒，无论做什么，都贵在坚持，我希望通过写博客能积累自己的技术与经验。不多说了，言归正传，下面开始介绍我是如何实现Cell标签自适应的吧。
image:
---
<p class="description"></p>

<!-- more -->

### 一、本地数据自适应
- 在做项目意见反馈的时候，需要选择反馈类型，整个界面是UITableView，我现在喜欢用自动布局，用的Masonry布局框架，开始选择类型是放在本地的，用Masonry实现cell高度自适应还算相对简单的，下面是实现数据在本地高度自适应的核心代码，该方法在cell初始化方法中调用：
```objective-c
- (void)initSubviews {
    /** << init subviews > */
    CGFloat margin = 15.f;
    CGFloat spacing = 10.f;
    CGFloat maxWidth = ScreenWidth;
    __block CGFloat rowWidth = 0;
    __block BOOL isNeedChangeLine = YES;
    __block UIButton *lastButton = nil;
    NSInteger count = self.dataArray.count;
    [self.dataArray enumerateObjectsUsingBlock:^(CYBImageTitleModel * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
        button.titleLabel.font = FONT(14.f);
        button.adjustsImageWhenHighlighted = NO;
        [button setTitleColor:[UIColor lightGrayColor]
                     forState:UIControlStateNormal];
        [button setTitleColor:Color_Orange
                     forState:UIControlStateSelected];
        [button setBackgroundImage:obj.image
                          forState:UIControlStateNormal];
        [button setBackgroundImage:obj.selectedImage
                          forState:UIControlStateSelected];
        [button setTitle:obj.title
                forState:UIControlStateNormal];
        button.tag = kBTN_TAG + idx;
        button.selected = obj.isSelected;
        if (obj.isSelected) {
            tempBtn = button;
        }
        [button wb_addTarget:self action:@selector(buttonClicked:)];
        
        [self.contentView addSubview:button];

        CGFloat titleWidth = [obj.title boundingRectWithSize:CGSizeMake(CGFLOAT_MAX, 28)
                                                     options:NSStringDrawingUsesLineFragmentOrigin
                                                  attributes:@{NSFontAttributeName : FONT(14.f)} context:nil].size.width + 2 * 8;
        rowWidth += titleWidth + spacing;
        /** < 是否需要换行 >  */
        if (rowWidth > maxWidth - 2 * margin) {
            isNeedChangeLine = YES;
            /** < 判断是否超过最大值 >  */
            if (titleWidth + 2 * margin > maxWidth) {
                titleWidth = maxWidth - 2 * margin;
            }
            /** < 换行后重新设置当前行的总宽度 >  */
            rowWidth = titleWidth + spacing;
        }
        
        [button mas_makeConstraints:^(MASConstraintMaker *make) {
            /** < 换行 >  */
            if (isNeedChangeLine) {
                if (!lastButton) {
                    make.top.equalTo(self.contentView.mas_top).offset(margin);
                }else {
                    make.top.equalTo(lastButton.mas_bottom).offset(spacing);
                }
                make.left.equalTo(self.contentView.mas_left).offset(margin);
                isNeedChangeLine = NO;
            }else {
                make.left.equalTo(lastButton.mas_right).offset(spacing);
                make.top.equalTo(lastButton.mas_top);
            }
            make.height.mas_equalTo(@(28));
            make.width.mas_equalTo(@(titleWidth));
            
            /** < 最后一个 >  */
            if (idx == count - 1) {
                make.bottom.equalTo(self.contentView.mas_bottom).offset(-margin);
            }
        }];
        lastButton = button;
    }];
}
```
### 二、网络请求数据高度自适应
- 后来改需求了，需要从网络请求意见反馈类型，好吧，上面的方法已经有实现高度自适应关键代码了，只要稍作修改就可实现了。但是实现过程并不是想象中那么简单，中间也经理了很多波折。因为时间还是很充裕的，我就考虑到将标签空间封装成一个视图，等要使用的时候自己添加到cell上，并设置上下左右约束，封装完成之后并没有达到我想要的效果，我发现cell根本就撑不起来，我检查了一遍约束，上下左右约束没有遗漏呀，封装的视图**WBAutoTagListView**核心代码如下，约束实在**layoutSubviews**设置的：
```objective-c
#pragma mark < Layout >
- (void)layoutSubviews {
    [super layoutSubviews];
    
    CGFloat maxWidth = self.bounds.size.width - _secionInset.left - _secionInset.right;
    __block CGFloat rowWidth = 0;
    __block BOOL isNeedChangeLine = YES;
    __block WBTagListItem *lastItem = nil;
    NSInteger count = self.itemArray.count;
    [self.itemArray enumerateObjectsUsingBlock:^(WBTagListItem * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        
        CGFloat titleWidth = obj.titleWidth;
        rowWidth += titleWidth + _minimumInteritemSpacing;
         /** < 是否需要换行 >  */
        if (rowWidth > maxWidth - 2 * _minimumInteritemSpacing) {
            isNeedChangeLine = YES;
            /** < 判断是否超过最大值 >  */
            if (titleWidth + 2 * _minimumInteritemSpacing > maxWidth) {
                titleWidth = maxWidth - 2 * _minimumInteritemSpacing;
            }
            /** < 换行后重新设置当前行的总宽度 >  */
            rowWidth = titleWidth + _minimumInteritemSpacing;
        }
        [obj mas_makeConstraints:^(MASConstraintMaker *make) {
            /** < 换行 >  */
            if (isNeedChangeLine) {
                if (!lastItem) {
                    make.top.equalTo(self.mas_top).offset(_secionInset.top);
                }else {
                    make.top.equalTo(lastItem.mas_bottom).offset(_minimumLineSpacing);
                }
                make.left.equalTo(self.mas_left).offset(_secionInset.left);
                isNeedChangeLine = NO;
            }else {
                make.left.equalTo(lastItem.mas_right).offset(_minimumInteritemSpacing);
                make.top.equalTo(lastItem.mas_top);
            }
            make.height.mas_equalTo(@(_itemHeight));
            make.width.mas_equalTo(@(titleWidth));
            
            /** < 最后一个 >  */
            if (idx == count - 1) {
                make.bottom.mas_offset(-_secionInset.bottom).priorityMedium();
            }
        }];
        lastItem = obj;
    }];
    NSLog(@"%f",[self systemLayoutSizeFittingSize:UILayoutFittingCompressedSize].height);
}
```
- 经测试，将该视图添加到控制器的视图上是可以自适应高度的，但是添加的cell上，就无法撑cell高度，尝试了许多写法，还是未能实现，控制台提示了无法算出cell的高度，就给了一个默认高度，顿时都无语了，有知道的大神能告诉我为什么有内容却无法撑起cell高度吗？
- 既然封装的视图无法实现cell高度自适应，我就尝试另外的思路方法，既然是cell自适应，那就索性封装一个标签cell吧**WBTagListViewCell**，为了可复用性，也为**WBTagListViewCell**添加了一些配置属性，如下：
```objective-c
/** < 数据源 > */
@property (nonatomic, strong) NSArray <WBTagListModel *>*items;
/** < 内边距 默认： UIEdgeInsetsMake(15, 15, 15, 15) > */
@property (nonatomic, assign) UIEdgeInsets secionInset;
/** < 行间距 默认：15 > */
@property (nonatomic, assign) CGFloat minimumLineSpacing;
/** < item之间距离 默认：10 > */
@property (nonatomic, assign) CGFloat minimumInteritemSpacing;
/** < 是否允许点击 默认：YES > */
@property (nonatomic, assign) BOOL allowSelection;
/** < 是否允许多选 默认：NO > */
@property (nonatomic, assign) BOOL allowMultipleSelection;
/** < 标签高度 默认：28.f > */
@property (nonatomic, assign) CGFloat itemHeight;

/** < 标签左右间距 默认：10.f > */
@property (nonatomic, assign) CGFloat leftRightMargin;
/** < 背景图片 > */
@property (nonatomic, copy) NSString *bgImageName;
/** < 选中背景图片 > */
@property (nonatomic, copy) NSString *selectedBgImageName;
/** < 标签颜色 默认：浅灰色 > */
@property (nonatomic, strong) UIColor *titleColor;
/** < 按钮选中颜色 > */
@property (nonatomic, strong) UIColor *titleSelectedColor;
/** < 标题大小 默认：14pt > */
@property (nonatomic, strong) UIFont *font;
/** < 边框宽度 默认：0 > */
@property (nonatomic, assign) CGFloat borderWidth;
/** < 边框颜色 bodoerWidth > 0 设置有效 > */
@property (nonatomic, strong) UIColor *borderColor;
/** < 选中边框颜色 bodoerWidth > 0 设置有效 > */
@property (nonatomic, strong) UIColor *selectedBorderColor;
/** < 圆角大小 默认：0 > */
@property (nonatomic, assign) CGFloat cornerRadius;
/** < 选中的item > */
@property (nonatomic, strong) NSMutableArray *selectedItems;

@property (nonatomic, weak) id <WBTagListViewCellDelegate> delegate;
```
- 关键实现步骤是重写了cell的**updateConstraints**，在有数据源的时候调用**setNeedsUpdateConstraints**，关键代码如下：
```objective-c
- (void)createTagWithData:(NSArray <WBTagListModel *>*)itemsArray {
    for (UIView *view in self.itemArray) {
        [view removeFromSuperview];
    }
    [self.itemArray removeAllObjects];
    
    for (NSInteger index = 0; index < itemsArray.count; index ++) {
        WBTagListItem *item = [WBTagListItem new];
        item.title = itemsArray[index].title;
        item.isSelected = itemsArray[index].isSelected;
        item.itemTag = index;
        item.delegate = self;
        [self.contentView addSubview:item];
        [self.itemArray addObject:item];
        
        /** < 默认选中第一个 > */
        if (index == 0 && itemsArray[index].isSelected) {
            _tempItem = item;
            [self.selectedItems removeAllObjects];
            [self.selectedItems addObject:_tempItem];
        }
    }
    [self setNeedsUpdateConstraints];
}

- (void)updateConstraints {
    [super updateConstraints];
    
    CGFloat maxWidth = self.contentView.bounds.size.width - _secionInset.left - _secionInset.right;
    __block CGFloat rowWidth = 0;
    __block BOOL isNeedChangeLine = YES;
    __block WBTagListItem *lastItem = nil;
    NSInteger count = self.itemArray.count;
    [self.itemArray enumerateObjectsUsingBlock:^(WBTagListItem * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        
        CGFloat titleWidth = obj.titleWidth;
        rowWidth += titleWidth + _minimumInteritemSpacing;
        /** < 是否需要换行 >  */
        if (rowWidth > maxWidth - 2 * _minimumInteritemSpacing) {
            isNeedChangeLine = YES;
            /** < 判断是否超过最大值 >  */
            if (titleWidth + 2 * _minimumInteritemSpacing > maxWidth) {
                titleWidth = maxWidth - 2 * _minimumInteritemSpacing;
            }
            /** < 换行后重新设置当前行的总宽度 >  */
            rowWidth = titleWidth + _minimumInteritemSpacing;
        }
        [obj mas_makeConstraints:^(MASConstraintMaker *make) {
            /** < 换行 >  */
            if (isNeedChangeLine) {
                if (!lastItem) {
                    make.top.equalTo(self.contentView.mas_top).offset(_secionInset.top);
                }else {
                    make.top.equalTo(lastItem.mas_bottom).offset(_minimumLineSpacing);
                }
                make.left.equalTo(self.contentView.mas_left).offset(_secionInset.left);
                isNeedChangeLine = NO;
            }else {
                make.left.equalTo(lastItem.mas_right).offset(_minimumInteritemSpacing);
                make.top.equalTo(lastItem.mas_top);
            }
            make.height.mas_equalTo(@(_itemHeight));
            make.width.mas_equalTo(@(titleWidth));
            
            /** < 最后一个 >  */
            if (idx == count - 1) {
                make.bottom.equalTo(self.contentView.mas_bottom).offset(-_secionInset.bottom).priorityMedium();
            }
        }];
        lastItem = obj;
    }];
}
```
- 最后运行效果也贴一张图吧
  ![Simulator Screen Shot - iPhone X - 2018-06-07 at 23.48.33.png](https://user-gold-cdn.xitu.io/2018/6/8/163db0915798250b?w=884&h=1915&f=png&s=173327)
- 封装cell在实现过程中，也遇到一些问题，最开始把约束写到**layoutSubviews**还是无法自适应高度，再就是要考虑到cell复用的问题。不管怎样最后还是实现了自己想要的效果，由于技术有限，可能我有写的不对不好的地方，还请斧正。最后也贴出自动布局和frame布局标签demo，如果觉得对你有帮助，请Star鼓励下吧。
### 三、GitHub Demo
Auto：[WBAutoTagListViewDemo](https://github.com/wenmobo/WBTagsView/tree/master/WBAutoTagListViewDemo)  
Frame：[WB_TagsViewDemo](https://github.com/wenmobo/WBTagsView/tree/master/WB_TagsViewDemo)

### 参考
- [Apple官方文档](https://developer.apple.com/documentation/uikit/uiview/1622450-setneedsupdateconstraints)