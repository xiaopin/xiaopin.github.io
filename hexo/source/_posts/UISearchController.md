---
title: UISearchController使用示例(push到下一个页面)
date: 2018-07-11 16:48:59
tags:
	- iOS
---

## GIF效果图

![GIF](/images/201807/UISearchController.gif)

## 源码

- FirstViewController，展示数据
```ObjC
@interface FirstViewController : UITableViewController<UISearchResultsUpdating>
{
    UISearchController *searchController;
    NSArray<NSString *> *titles;
}
@end

@implementation FirstViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    self.title = @"UISearchController";
    SearchResultViewController *resultVC = [[SearchResultViewController alloc] initWithStyle:UITableViewStylePlain];
    searchController = [[UISearchController alloc] initWithSearchResultsController:resultVC];
    searchController.searchResultsUpdater = self;
    searchController.searchBar.keyboardType = UIKeyboardTypeNumberPad;
    self.tableView.tableHeaderView = searchController.searchBar;
    self.definesPresentationContext = YES; // 这是push成功的关键
    
    NSMutableArray<NSString *> *temp = [NSMutableArray array];
    for (int i=0; i<40; i++) {
        NSString *title = [NSString stringWithFormat:@"%d-%u", i,arc4random()];
        [temp addObject:title];
    }
    titles = [NSArray arrayWithArray:temp];
}

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    return titles.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    static NSString * const identifier = @"Cell";
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:identifier];
    if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:identifier];
        cell.accessoryType = UITableViewCellAccessoryDisclosureIndicator;
    }
    cell.textLabel.text = titles[indexPath.row];
    return cell;
}

- (void)updateSearchResultsForSearchController:(UISearchController *)searchController {
    NSString *keyword = searchController.searchBar.text;
    NSPredicate *predicate = [NSPredicate predicateWithFormat:@"SELF CONTAINS[c] %@", keyword];
    NSArray<NSString *> *results = [titles filteredArrayUsingPredicate:predicate];

    SearchResultViewController *resultVC = (SearchResultViewController*)searchController.searchResultsController;
    resultVC.resultTitles = results;
    [resultVC.tableView reloadData];
    
}

@end
```

- SearchResultViewController 源码，用于展示搜索结果内容
```ObjC
@interface SearchResultViewController : UITableViewController

@property (nonatomic, strong) NSArray<NSString*> *resultTitles;

@end

@implementation SearchResultViewController

- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    return self.resultTitles.count;
}

- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    static NSString * const identifier = @"Cell";
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:identifier];
    if (cell == nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:identifier];
        cell.accessoryType = UITableViewCellAccessoryDisclosureIndicator;
    }
    cell.textLabel.text = self.resultTitles[indexPath.row];
    return cell;
}

- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath {
    [tableView deselectRowAtIndexPath:indexPath animated:YES];
    // 点击搜索结果内容push到下一个页面
    UIViewController *vc = [[UIViewController alloc] init];
    vc.view.backgroundColor = [UIColor brownColor];
    [self.presentingViewController.navigationController pushViewController:vc animated:YES];
}

@end
```

- AppDelegate
```ObjC
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    FirstViewController *firstVC = [[FirstViewController alloc] initWithStyle:UITableViewStylePlain];
    UINavigationController *nav = [[UINavigationController alloc] initWithRootViewController:firstVC];
    
    self.window = [[UIWindow alloc]initWithFrame:[UIScreen mainScreen].bounds];
    self.window.rootViewController = nav;
    [self.window makeKeyAndVisible];

    return YES;
}
```

## 说明

1、`definesPresentationContext` 属性必须设置为`YES`，否则跳转不成功或者跳转后新页面处于搜索结果页面之下
```ObjC
self.definesPresentationContext = YES;
```

2、正确的 push 姿势，使用 `self.presentingViewController.navigationController` 进行跳转
```ObjC
[self.presentingViewController.navigationController pushViewController:vc animated:YES];
```

## 2018-10-25 更新

> 场景：C (UISearchController) 作为控制器 A (UIViewController) 的成员属性，B (UIViewController) 作为 C 的 searchResultsController。

如果 `hidesNavigationBarDuringPresentation` 属性设置为 `NO` 并且 UISearchController 处于激活状态（`isActive` 属性为 YES），如果此时通过导航栏返回上一个页面（不管是点击导航栏返回按钮或是通过滑动手势返回上一页）将会导致 `searchResultsController` 不会被释放，也就是内存泄漏了。对应于上面的场景就是 A、B、C 都不会被释放。

那么如何解决这个问题呢，可以重写 A 的 `viewDidDisappear:` 方法，在 A 消失的时候进行处理，如果此时 UISearchController 处于激活状态，只需要关闭激活状态即可。

```ObjC
- (void)viewDidDisappear:(BOOL)animated {
    [super viewDidDisappear:animated];
    if (self.searchController.isActive && ![self.navigationController.viewControllers containsObject:self]) {
        [self.searchController setActive:NO];
    }
}
```
