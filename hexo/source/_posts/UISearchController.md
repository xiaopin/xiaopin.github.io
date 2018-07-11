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


