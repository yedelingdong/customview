1. recyclerview分析

入口：
recyclerview里面的onTouchEvent方法的ACTION_MOVE事件 -> scrollByInternal -> scrollStep -> mLayout.scrollVerticallyBy -> scrollBy
  -> fill -> layoutChunk -> layoutState.next (这个过程在下面单独分析) -> addView

layoutState.next -> getViewForPosition -> tryGetViewHolderForPositionByDeadline

tryGetViewHolderForPositionByDeadline 取 ViewHolder
1. getChangedScrapViewForPosition -> mChangedScrap 与动画相关
2.1 getScrapOrHiddenOrCachedHolderForPosition -> mAttachedScrap,mCachedViews
2.2 getScrapOrCachedViewForId -> mAttachedScrap,mCachedViews (ViewType,itemid)
3. getChildViewHolder -> mViewCacheExtension.getViewForPositionAndType 自定义缓存
4. getRecycledViewPool -> mRecyclerPool从缓冲池里面获取

mChangedScrap: ArrayList<ViewHolder> , 缓存还在屏幕内的ViewHolder
mAttachedScrap: ArrayList<ViewHolder>, 缓存还在屏幕内的ViewHolder
mCachedViews: ArrayList<ViewHolder>, 缓存移除屏幕之外的ViewHolder
mViewCacheExtension: abstract static class, 提供给用户的自定义扩展缓存，需要用户自己管理view的创建和缓存
mRecyclerPool: SparseArray<ScrapData>, static class ScrapData定义了ArrayList<ViewHolder> mScrapHeap，ViewHolder缓存池

ViewHolder -> 包装View -> itemView

多级缓存的目的 -> 为了性能
glide也实现了多级缓存

如果缓存中没有ViewHolder则创建
createViewHolder -> onCreateViewHolder创建ViewHolder

tryBindViewHolderByDeadline -> mAdapter.bindViewHolder -> onBindViewHolder


问题：
1. recyclerview回收复用的是谁？
A：ViewHolder

2. recyclerview有几级缓存？
A：4级。第一级：mChangedScrap，第二级：mAttachedScrap,mCachedViews，第三级：mViewCacheExtension，第四级：mRecyclerPool

3. 如何实现自定义LayoutManager
A：继承RecyclerView.LayoutManager，重写generateDefaultLayoutParams和onLayoutChildren方法

4. 回收什么？复用什么？
A：回收ArrayList里面的ViewHolder，复用是从ArrayList里面取

5. 回收到哪里去？从哪里获得复用？
A: 回收到ArrayList<ViewHolder>.

6：什么时候缓存？什么时候复用？
A: onLayoutChildren -> detachAndScrapAttachedViews -> scrapOrRecycleView
  -> recycleViewHolderInternal
      -> recycleCachedViewAt -> addViewHolderToRecycledViewPool -> getRecycledViewPool().putRecycledView
      -> scrapHeap.add -> mCachedViews.remove(这里要删除)
      -> mCachedViews.add
      -> addViewHolderToRecycledViewPool
  -> recycler.scrapView -> mAttachedScrap.add
                        -> mChangedScrap.add

