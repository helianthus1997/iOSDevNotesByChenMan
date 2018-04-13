iOS·长按保存图片到相册：系统原生UIActionSheet与UIAlertView，UIAlertController等方案


> 场景: 在一个VC中，为一个UICollectionViewCell中的图片添加长按图片保存的事件。

![长按保存图片](https://upload-images.jianshu.io/upload_images/1283539-372d06523e4bec9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 前提：infoPlist中添加相应权限：Privacy - Photo Library Additions Usage Description。否则进行保存图片的时候APP会奔溃。

![image.png](https://upload-images.jianshu.io/upload_images/1283539-3f823a46d0297f9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- VC背景，及需要遵守的代理
```
@interface CustomerImageViewController ()<UICollectionViewDataSource,UICollectionViewDelegate,UIActionSheetDelegate>

@property (weak, nonatomic) IBOutlet UICollectionView *collectionView;
@property (weak, nonatomic) IBOutlet UICollectionViewFlowLayout *flowLayout;
@property (weak, nonatomic) IBOutlet UILabel *pageLabel;

@property (nonatomic, strong) NSMutableArray<NSString *> *imageUrlStrArr;
@property (nonatomic, strong) UIImage *tempImage;
```
# 1. UIActionSheet实现底部弹框


- 给CollectionViewCell中的UIImageView添加事件
```
#pragma - mark - UICollectionViewDataSource
- (NSInteger)collectionView:(UICollectionView *)collectionView numberOfItemsInSection:(NSInteger)section{
    return self.imageUrlStrArr.count;
}

- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView cellForItemAtIndexPath:(NSIndexPath *)indexPath{
    
    MyCollectionViewCell *cell = [collectionView dequeueReusableCellWithReuseIdentifier:NSStringFromClass([MyCollectionViewCell class]) forIndexPath:indexPath];
    //传数据
    [cell setImageUrlStr:self.imageUrlStrArr[indexPath.item]];
    //事件
    UILongPressGestureRecognizer *ges = [[UILongPressGestureRecognizer alloc]initWithTarget:self action:@selector(longPressAction:)];
    [cell.imageView addGestureRecognizer:ges];
    return cell;
}

-(void)longPressAction:(UILongPressGestureRecognizer*)gesture {
    if(gesture.state == UIGestureRecognizerStateBegan)
    {
        UIActionSheet *actionSheet = [[UIActionSheet alloc]initWithTitle:nil delegate:self cancelButtonTitle:@"取消"destructiveButtonTitle:nil otherButtonTitles:@"保存图片",nil];

        actionSheet.actionSheetStyle = UIActionSheetStyleBlackOpaque;
        [actionSheet showInView:self.view];
        
        UIImageView *imgView = (UIImageView*)[gesture view];
        _tempImage = imgView.image;
    }
}
```
- 实现UIActionSheetDelegate代理方法
```
#pragma - mark - UIActionSheetDelegate
- (void)actionSheet:(UIActionSheet*)actionSheet didDismissWithButtonIndex:  (NSInteger)buttonIndex
{
    if(buttonIndex ==0) {
        if(_tempImage){
            UIImageWriteToSavedPhotosAlbum(_tempImage,self,@selector(imageSavedToPhotosAlbum:didFinishSavingWithError:contextInfo:),nil);
        }else{
            [Toast showCenterWithText:@"保存失败:没有网络"];
        }
    }
}

- (void)imageSavedToPhotosAlbum:(UIImage*)image didFinishSavingWithError: (NSError*)error contextInfo:(void*)contextInfo
{
    NSString*message =@"";
    
    if(!error) {
        
        message =@"成功保存到相册";
        
        UIAlertView*alert = [[UIAlertView alloc]initWithTitle:@"提示"message:message delegate:self cancelButtonTitle:@"确定"otherButtonTitles:nil];
        
        [alert show];
        
    }else{
        message = [error description];
        
        UIAlertView*alert = [[UIAlertView alloc]initWithTitle:@"提示"message:message delegate:self cancelButtonTitle:@"确定"otherButtonTitles:nil];
        
        [alert show];
        
    }
}
```

![长按保存事件](https://upload-images.jianshu.io/upload_images/1283539-1c7c1cefbfb18f21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![保存成功提示](https://upload-images.jianshu.io/upload_images/1283539-a9d2787a116b4661.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 2. UIAlertView实现中部弹框

- 修改点1：VC遵守的协议
```
@interface CustomerImageViewController ()<UICollectionViewDataSource,UICollectionViewDelegate, UIAlertViewDelegate >
```

- 修改点2：VC添加两个属性
```
@property (nonatomic , strong) UIAlertView *myAlertView;
@property (nonatomic , strong) UIAlertView *myAlertView2;

```
- 修改点3：longPressAction方法的实现
```
-(void)longPressAction:(UILongPressGestureRecognizer*)gesture {
    if(gesture.state == UIGestureRecognizerStateBegan)
    {
        UIImageView *imgView = (UIImageView*)[gesture view];
        _tempImage = imgView.image;
        
        self.myAlertView = [[UIAlertView alloc] initWithTitle:@"提示" message:@"您要保存当前图片到相册中吗？" delegate:self cancelButtonTitle:@"取消" otherButtonTitles:@"保存",nil];
        [self.myAlertView show];
    }
}
```
- 修改点4：实现UIAlertViewDelegate代理方法

```
#pragma - mark -  UIAlertViewDelegate 
- (void)alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex
{
    if (buttonIndex == 1) {
        // 保存照片(获取到点击的 image)
        //        NSInteger i = self.scroll.contentOffset.x / self.scroll.bounds.size.width;
        if (_tempImage) {
            UIImageWriteToSavedPhotosAlbum(_tempImage, self, @selector(image:didFinshSavingWithError:contextInfo:), NULL);
        }
    }
}

// 保存图片错误提示方法
- (void)image:(UIImage *)image didFinshSavingWithError:(NSError *)error contextInfo:(void *)contextInfo
{
    NSString *mes = nil;
    if (error != nil) {
        mes = @"保存图片失败";
    } else {
        mes = @"保存图片成功";
    }
    self.myAlertView2 = [[UIAlertView alloc] initWithTitle:@"提示" message:mes delegate:self cancelButtonTitle:nil otherButtonTitles:nil];
    [self.myAlertView2 show];
    [NSTimer scheduledTimerWithTimeInterval:0.8f target:self selector:@selector(performDismiss:) userInfo:nil repeats:NO];
}

- (void)performDismiss:(NSTimer *)timer {
    [self.myAlertView2 dismissWithClickedButtonIndex:0 animated:YES];
}

```

![长按保存图片](https://upload-images.jianshu.io/upload_images/1283539-9cbe2d23918daf0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 3. UIAlertController实现底部/中部弹框

> 下面的修改点是针对上面的第二节2. UIAlertView的代码

- 修改点1：longPressAction方法的实现

```
-(void)longPressAction:(UILongPressGestureRecognizer*)gesture {
    if(gesture.state == UIGestureRecognizerStateBegan)
    {
        UIImageView *imgView = (UIImageView*)[gesture view];
        _tempImage = imgView.image;
        
        [self handleActionSheet];
    }
}

```


- 修改点2：实现如上的handleActionSheet（添加UIAlertController）

```
- (void)handleActionSheet{
    
    UIAlertController *actionSheetController = [UIAlertController alertControllerWithTitle:nil message:nil preferredStyle:UIAlertControllerStyleActionSheet];
    
    UIAlertAction *saveImageAction = [UIAlertAction actionWithTitle:@"保存图片" style:UIAlertActionStyleDefault handler:^(UIAlertAction * _Nonnull action) {
        if (_tempImage) {
            UIImageWriteToSavedPhotosAlbum(_tempImage, self, @selector(image:didFinshSavingWithError:contextInfo:), NULL);
        }
    }];

    UIAlertAction *cancelAction = [UIAlertAction actionWithTitle:@"取消" style:UIAlertActionStyleCancel handler:nil];
    
    [actionSheetController addAction:cancelAction];
    [actionSheetController addAction:saveImageAction];
    
    [self presentViewController:actionSheetController animated:YES completion:nil];
}
```
> 其中，把**preferredStyle**的参数**UIAlertControllerStyleActionSheet**换成**UIAlertControllerStyleAlert**，就是中部弹框了。

- 相同点1：成功及错误处理

```
// 保存图片错误提示方法
- (void)image:(UIImage *)image didFinshSavingWithError:(NSError *)error contextInfo:(void *)contextInfo
{
    NSString *mes = nil;
    if (error != nil) {
        mes = @"保存图片失败";
    } else {
        mes = @"保存图片成功";
    }
    self.myAlertView2 = [[UIAlertView alloc] initWithTitle:@"提示" message:mes delegate:self cancelButtonTitle:nil otherButtonTitles:nil];
    [self.myAlertView2 show];
    [NSTimer scheduledTimerWithTimeInterval:0.8f target:self selector:@selector(performDismiss:) userInfo:nil repeats:NO];
}
```

![长按保存图片](https://upload-images.jianshu.io/upload_images/1283539-372d06523e4bec9e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


