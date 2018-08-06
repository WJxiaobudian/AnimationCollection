# AnimationCollection

重写 UICollectionViewFlowLayout
- (void)prepareLayout {
    [super prepareLayout];
    self.scrollDirection = UICollectionViewScrollDirectionHorizontal;
    CGFloat width = (self.collectionView.frame.size.width - self.itemSize.width)/2;
    self.sectionInset = UIEdgeInsetsMake(0, width, 0, width);
}

/*
 当collectionView 显示范围改变时，是否需要重新刷新布局
 即重新处理布局属性
 
 */

- (BOOL)shouldInvalidateLayoutForBoundsChange:(CGRect)newBounds {
//    return NO;
    return YES;
}

/**
 返回的数组(存放rect范围内所有元素的布局属性)
 这个方法返回值决定rect范围内所有元素的排布(frame)
 
 */
- (NSArray<UICollectionViewLayoutAttributes *> *)layoutAttributesForElementsInRect:(CGRect)rect {
    NSArray *array = [[NSArray alloc] initWithArray:[super layoutAttributesForElementsInRect:rect] copyItems:YES];
    // 屏幕中线
    CGFloat centenX = self.collectionView.contentOffset.x + self.collectionView.bounds.size.width/2.0f;
    // 刷新cell缩放
    for (UICollectionViewLayoutAttributes *attributes in array) {
        CGFloat distance = fabs(attributes.center.x - centenX);
        // 移动的距离和屏幕宽度的比例
        CGFloat apartScale = distance/self.collectionView.bounds.size.width;
        // 把卡片移动范围固定到 -π/4 到 +π/4 这一个范围内
        CGFloat scale = fabs(cos(apartScale * M_PI/6));
       // 设置cell的缩放 按照余弦函数曲线 越居中越趋近于1
        if (scale > 1) {
            scale = 1;
        }
        attributes.transform = CGAffineTransformMakeScale(1.0, scale);
    }
    return  array;
}

/**
 这个方法决定滚动结束时的偏移量
 */
- (CGPoint)targetContentOffsetForProposedContentOffset:(CGPoint)proposedContentOffset withScrollingVelocity:(CGPoint)velocity {
    CGFloat centenX = proposedContentOffset.x + self.collectionView.frame.size.width * 0.5;
    CGRect rect;
    rect.origin.x = proposedContentOffset.x;
    rect.origin.y = 0;
    rect.size = self.collectionView.frame.size;
    
    NSArray *array = [[NSArray alloc] initWithArray:[super layoutAttributesForElementsInRect:rect] copyItems:YES];
    
    CGFloat mindalta = MAXFLOAT;
    for (UICollectionViewLayoutAttributes * attrs in array) {
        if (ABS(mindalta) > ABS(attrs.center.x - centenX)) {
            mindalta = attrs.center.x - centenX;
        }
    }
    CGFloat s = proposedContentOffset.x + mindalta;
    // 这里处理数据精度的问题
    if (s < 0 && s > -0.0000000001) {
        s = 0;
    }
    proposedContentOffset.x = s;
    return proposedContentOffset;
}
