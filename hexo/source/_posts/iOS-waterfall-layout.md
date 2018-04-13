---
title: UICollectionView瀑布流布局
date: 2018-04-10 16:28:58
tags:
	- iOS
	- Swift
---

> 仅作为学习交流之用

### 环境

- Xcode9.0+
- Swift4.0+

### TODO

- 废弃`CollectionViewWaterfallLayout`的`contentInset`属性，改用UICollectionView的`contentInset`属性。

一开始是采用`UICollectionView.contentInset`来做四周留白的，但是后来发现一旦设置了`UICollectionView.contentInset.left`的值且`>0`的时候，整个UICollectionView就会一片空白，但是top、bottom、right属性均不受此影响。测试了网上的一些Demo也发现存在此问题，具体原因不详，如有知道的还望赐教。

鉴于此，暂时给CollectionViewWaterfallLayout增加了一个contentInset属性。

### 源码

```Swift
protocol CollectionViewDelegateWaterfallLayout {
    
    /// 返回每个cell的高度
    func collectionView(_ collectionView: UICollectionView, layout collectionViewLayout: UICollectionViewLayout, heightForItemAt indexPath: IndexPath, itemWidth width: CGFloat) -> CGFloat
    
}


class CollectionViewWaterfallLayout: UICollectionViewLayout {

    /// 纵列数量
    var column: Int = 2
    /// 每一行cell之间的间距
    var minimumLineSpacing: CGFloat = 5.0
    /// 每一纵列cell之间的间距
    var minimumInteritemSpacing: CGFloat = 5.0
    ///
    var delegate: CollectionViewDelegateWaterfallLayout?
    /// 内容偏移
    var contentInset: UIEdgeInsets = .zero
    /// 每个纵列的Y值偏移量
    private var maxYForColumn = [CGFloat]()
    /// 布局数据
    private var layoutAttributes = [UICollectionViewLayoutAttributes]()
    
    override var collectionViewContentSize: CGSize {
        guard let max = maxYForColumn.max() else { return .zero }
        return CGSize(width: 0.0, height: max + contentInset.bottom)
    }
    
    override func prepare() {
        super.prepare()
        maxYForColumn = Array(repeating: contentInset.top, count: column)
        layoutAttributes.removeAll()
        guard let collectionView = collectionView else { return }
        guard let delegate = delegate else {
            assertionFailure("delegate can'not be nil.")
            return
        }
        let contentWidth = collectionView.bounds.width - contentInset.left - contentInset.right
        let width = (contentWidth - CGFloat(column - 1) * minimumInteritemSpacing) / CGFloat(column)
        let itemCount = collectionView.numberOfItems(inSection: 0)
        for item in 0..<itemCount {
            let indexPath = IndexPath(item: item, section: 0)
            let height = delegate.collectionView(collectionView, layout: self, heightForItemAt: indexPath, itemWidth: width)
            // 计算当前cell应该展示在哪一列上
            var targetColumn: Int
            if item < column {
                targetColumn = item
            } else {
                let min = maxYForColumn.min()!
                targetColumn = maxYForColumn.index(of: min)!
            }
            // 计算frame
            let x = contentInset.left + width * CGFloat(targetColumn) + CGFloat(targetColumn) * minimumInteritemSpacing
            let y = minimumLineSpacing + maxYForColumn[targetColumn]
            let attribute = UICollectionViewLayoutAttributes(forCellWith: indexPath)
            attribute.frame = CGRect(x: x, y: y, width: width, height: height)
            layoutAttributes.append(attribute)
            // 更新当前列的Y值偏移量
            maxYForColumn[targetColumn] = y + height
        }
    }
    
    override func layoutAttributesForItem(at indexPath: IndexPath) -> UICollectionViewLayoutAttributes? {
        return layoutAttributes[indexPath.item]
    }
    
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        return layoutAttributes
    }
    
    override func shouldInvalidateLayout(forBoundsChange newBounds: CGRect) -> Bool {
        guard let collectionView = collectionView else { return false }
        return !collectionView.bounds.size.equalTo(newBounds.size)
    }
    
}
```
