#####导航栏背景透明 
```
[self.navigationController.navigationBar setBackgroundImage:[UIImage new] forBarMetrics:UIBarMetricsDefault];
```
#####导航栏底部线清除
```
self.navigationController.navigationBar.barStyle = UIBarStyleBlack; 
self.navigationController.navigationBar.translucent = YES; 
[self.navigationController.navigationBar setShadowImage:[UIImage new]];
```
#####导航栏底部线清除
```
UIView *statusBarView = [[UIView alloc] initWithFrame:CGRectMake(0, -20, ScreenWidth, 20)];
statusBarView.backgroundColor = HRMainColorGreen;
[self.navigationBar addSubview:statusBarView];
```
    


