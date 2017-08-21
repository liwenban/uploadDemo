#uploadDemo

#前言：
> 对于Alamofire文件上传，网上大多数的教程的Alamofire版本都是在3.0。但是Alamofire在4.0的版本上作出了重大更新，特别在文件上传出现了很多变化，网上的代码直接copy会有错误。另外，网上的大部分代码并不提供后台，对于并不了解文件后台接收的swift学习爱好者来说是头疼的。其次，对于视频上传的网上中文资料少之又少，大多数都是一些图片（小文件）上传，没有尝试上传视频这样的大文件。闲暇之时写一篇关于Alamofire最新的文件上传的教程，希望大家喜欢。

#Demo
![上传图片](http://upload-images.jianshu.io/upload_images/6327935-b66374dd4323a064.gif?imageMogr2/auto-orient/strip)

![上传视频](http://upload-images.jianshu.io/upload_images/6327935-54d606a0a7aa9fb0.gif?imageMogr2/auto-orient/strip)

![服务器文件](http://upload-images.jianshu.io/upload_images/6327935-3b8f7e517201dc65.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#客户端
`Info.plist`

![QQ20170821-234530.png](http://upload-images.jianshu.io/upload_images/6327935-76ec2efed7e41cb4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
`ViewController.swift`
```
//
//  ViewController.swift
//  uploadDemo
//
//  Created by liwenban on 2017/8/21.
//  Copyright © 2017年 hellomiao. All rights reserved.
//

import UIKit
import AVFoundation
import MobileCoreServices
import AssetsLibrary
import AVKit
import Alamofire
import MediaPlayer
import SwiftyJSON

class ViewController: UIViewController,UIImagePickerControllerDelegate, UINavigationControllerDelegate {

//设置标志，用于标识上传那种类型文件（图片／视频）
var flag = ""
//设置服务器地址
let uploadURL = "http://www.hellomiao.cn/hellomiao/upload.php"

override func viewDidLoad() {
super.viewDidLoad()

}

//按钮事件：上传图片
@IBAction func uploadImage(_ sender: Any) {
photoLib()
}

//按钮事件：上传视频
@IBAction func uploadVideo(_ sender: Any) {
videoLib()
}

//图库 - 照片
func photoLib(){
//
flag = "图片"
//判断设置是否支持图片库
if UIImagePickerController.isSourceTypeAvailable(.photoLibrary){
//初始化图片控制器
let picker = UIImagePickerController()
//设置代理
picker.delegate = self
//指定图片控制器类型
picker.sourceType = UIImagePickerControllerSourceType.photoLibrary
//弹出控制器，显示界面
self.present(picker, animated: true, completion: {
() -> Void in
})
}else{
print("读取相册错误")
}
}


//图库 - 视频
func videoLib(){
flag = "视频"
if UIImagePickerController.isSourceTypeAvailable(.photoLibrary) {
//初始化图片控制器
let imagePicker = UIImagePickerController()
//设置代理
imagePicker.delegate = self
//指定图片控制器类型
imagePicker.sourceType = .photoLibrary;
//只显示视频类型的文件
imagePicker.mediaTypes =  [kUTTypeMovie as String]
//不需要编辑
imagePicker.allowsEditing = false
//弹出控制器，显示界面
self.present(imagePicker, animated: true, completion: nil)
}
else {
print("读取相册错误")
}
}

//选择成功后代理
func imagePickerController(_ picker: UIImagePickerController,didFinishPickingMediaWithInfo info: [String : Any]) {
if flag == "视频" {

//获取选取的视频路径
let videoURL = info[UIImagePickerControllerMediaURL] as! URL
let pathString = videoURL.path
print("视频地址：\(pathString)")
//图片控制器退出
self.dismiss(animated: true, completion: nil)
let outpath = NSHomeDirectory() + "/Documents/\(Date().timeIntervalSince1970).mp4"
//视频转码
self.transformMoive(inputPath: pathString, outputPath: outpath)
}else{
//flag = "图片"

//获取选取后的图片
let pickedImage = info[UIImagePickerControllerOriginalImage] as! UIImage
//转成jpg格式图片
guard let jpegData = UIImageJPEGRepresentation(pickedImage, 0.5) else {
return
}
//上传
self.uploadImage(imageData: jpegData)
//图片控制器退出
self.dismiss(animated: true, completion:nil)
}
}

//上传图片到服务器
func uploadImage(imageData : Data){
Alamofire.upload(
multipartFormData: { multipartFormData in
//采用post表单上传
// 参数解释：
//withName:和后台服务器的name要一致 fileName:自己随便写，但是图片格式要写对 mimeType：规定的，要上传其他格式可以自行百度查一下
multipartFormData.append(imageData, withName: "file", fileName: "123456.jpg", mimeType: "image/jpeg")
//如果需要上传多个文件,就多添加几个
//multipartFormData.append(imageData, withName: "file", fileName: "123456.jpg", mimeType: "image/jpeg")
//......

},to: uploadURL,encodingCompletion: { encodingResult in
switch encodingResult {
case .success(let upload, _, _):
//连接服务器成功后，对json的处理
upload.responseJSON { response in
//解包
guard let result = response.result.value else { return }
print("\(result)")
//须导入 swiftyJSON 第三方框架，否则报错
let success = JSON(result)["success"].int ?? -1
if success == 1 {
print("上传成功")
let alert = UIAlertController(title:"提示",message:"上传成功", preferredStyle: .alert)
let action2 = UIAlertAction(title: "关闭", style: .default, handler: nil)
alert.addAction(action2)
self.present(alert , animated: true , completion: nil)
}else{
print("上传失败")
let alert = UIAlertController(title:"提示",message:"上传失败", preferredStyle: .alert)
let action2 = UIAlertAction(title: "关闭", style: .default, handler: nil)
alert.addAction(action2)
self.present(alert , animated: true , completion: nil)
}
}
//获取上传进度
upload.uploadProgress(queue: DispatchQueue.global(qos: .utility)) { progress in
print("图片上传进度: \(progress.fractionCompleted)")
}
case .failure(let encodingError):
//打印连接失败原因
print(encodingError)
}
})
}

//上传视频到服务器
func uploadVideo(mp4Path : URL){
Alamofire.upload(
//同样采用post表单上传
multipartFormData: { multipartFormData in
multipartFormData.append(mp4Path, withName: "file", fileName: "123456.mp4", mimeType: "video/mp4")
//服务器地址
},to: uploadURL,encodingCompletion: { encodingResult in
switch encodingResult {
case .success(let upload, _, _):
//json处理
upload.responseJSON { response in
//解包
guard let result = response.result.value else { return }
print("\(result)")
//须导入 swiftyJSON 第三方框架，否则报错
let success = JSON(result)["success"].int ?? -1
if success == 1 {
print("上传成功")
let alert = UIAlertController(title:"提示",message:"上传成功", preferredStyle: .alert)
let action2 = UIAlertAction(title: "关闭", style: .default, handler: nil)
alert.addAction(action2)
self.present(alert , animated: true , completion: nil)
}else{
print("上传失败")
let alert = UIAlertController(title:"提示",message:"上传失败", preferredStyle: .alert)
let action2 = UIAlertAction(title: "关闭", style: .default, handler: nil)
alert.addAction(action2)
self.present(alert , animated: true , completion: nil)
}
}
//上传进度
upload.uploadProgress(queue: DispatchQueue.global(qos: .utility)) { progress in
print("视频上传进度: \(progress.fractionCompleted)")
}
case .failure(let encodingError):
print(encodingError)
}
})
}

/// 转换视频
///
/// - Parameters:
///   - inputPath: 输入url
///   - outputPath:输出url
func transformMoive(inputPath:String,outputPath:String){


let avAsset:AVURLAsset = AVURLAsset(url: URL.init(fileURLWithPath: inputPath), options: nil)
let assetTime = avAsset.duration

let duration = CMTimeGetSeconds(assetTime)
print("视频时长 \(duration)");
let compatiblePresets = AVAssetExportSession.exportPresets(compatibleWith: avAsset)
if compatiblePresets.contains(AVAssetExportPresetLowQuality) {
let exportSession:AVAssetExportSession = AVAssetExportSession.init(asset: avAsset, presetName: AVAssetExportPresetMediumQuality)!
let existBool = FileManager.default.fileExists(atPath: outputPath)
if existBool {
}
exportSession.outputURL = URL.init(fileURLWithPath: outputPath)


exportSession.outputFileType = AVFileTypeMPEG4
exportSession.shouldOptimizeForNetworkUse = true;
exportSession.exportAsynchronously(completionHandler: {

switch exportSession.status{

case .failed:

print("失败...\(String(describing: exportSession.error?.localizedDescription))")
break
case .cancelled:
print("取消")
break;
case .completed:
print("转码成功")
//转码成功后获取视频视频地址
let mp4Path = URL.init(fileURLWithPath: outputPath)
//上传
self.uploadVideo(mp4Path: mp4Path)
break;
default:
print("..")
break;
}
})
}
}

}
```

#后台代码
`upload.php`
```
<?php
header('Content-type: text/json; charset=UTF-8' );

$response = array();
// 文件类型限制  "file"名字必须和iOS客户端上传的name一致
if ($_FILES["file"]["error"] > 0) {
$response ["error"] = "错误代码".$_FILES["file"]["error"];
echo json_encode($response);
} else {
//获取传入的文件名
$fillName = $_FILES['file']['name'];
//以 "." 为界对文件名分割，并存入数组
$dotArray = explode('.', $fillName);
//获取文件格式
$type = end($dotArray);
// - - - - -
//小技巧:客户端传入的文件名，除了文件的格式要对之外，文件名部分是可以随意填写。
//经过点分割后，就可以将文件名和文件格式分开，分开后下标为0的就是文件名，部分，这时候就间接实现了传参，获得用户的id
$userId = $dotArray[0];
// - - - -
//设置存入的文件名（路径）
$path = "../res/".$userId.'.'.$type;
// 从临时目录复制到目标目录
move_uploaded_file($_FILES["file"]["tmp_name"],$path);
//校验传入后文件是否存在
if (file_exists($path)){
$response ['success']= 1;
//json格式返回
echo json_encode($response);
}else {
$response ['success'] = 0;
echo json_encode($response);
}
}
?>
```

#注意事项
> 服务器是php环境的web服务器。php对上传文件的大小、超时时间有限制。因此再测试上传视频的时候，会出现上传失败是正常现象。

解决办法：
`php.ini`
```
可以通过 “command + F”搜索以下字段：
file_uploads = on ;是否允许通过HTTP上传文件的开关。默认为ON即是开
upload_tmp_dir ;文件上传至服务器上存储临时文件的地方，如果没指定就会用系统默认的临时文件夹
upload_max_filesize = 8m ;望文生意，即允许上传文件大小的最大值。默认为2M
post_max_size = 8m ;指通过表单POST给PHP的所能接收的最大值，包括表单里的所有值。默认为8M
一般地，设置好上述四个参数后，上传<=8M的文件是不成问题，在网络正常的情况下。
但如果要上传>8M的大体积文件，只设置上述四项还一定能行的通。

近一步配置以下的参数
max_execution_time = 600 ;每个PHP页面运行的最大时间值(秒)，默认30秒
max_input_time = 600 ;每个PHP页面接收数据所需的最大时间，默认60秒
memory_limit = 8m ;每个PHP页面所吃掉的最大内存，默认8M
把上述参数修改后，在网络所允许的正常情况下，就可以上传大体积文件了
max_execution_time = 600
max_input_time = 600
memory_limit = 32m
file_uploads = on
upload_tmp_dir = /tmp
upload_max_filesize = 32m
post_max_size = 32m
```
