jsp+servlet实战酷炫博客+聊天系统
=

``
如需源码请联系程序帮，QQ1022287044
``

项目介绍
----
> 本项目使用jsp+servlet+mysql架构搭建可聊天的酷炫博客系统。界面非常好看，除了登录注册个人中心修改外，博客还添加背景音乐，可在线交友聊天，发表动态，相互评论等，喜欢的博文还能添加收藏。



开发环境：
-----
1. jdk 8
2. intellij idea
3. tomcat 8
4. mysql 5.7

所用技术：
-----
1. jsp+servlet
2. js+ajax
3. layui
4. jdbc+C3P0

博客访问地址
---
```
http://localhost:8090/indexServlet
```

项目目录结构
-----

![项目结构](/image/代码架构.png)

运行效果
----

- 登录

![登录](/image/登录.png)

- 首页 

![首页](/image/主页.png)

- 动态评论

![动态评论](/image/评论.png)

- 关注列表

![关注列表](/image/我的列表.png)

- 个人资料

![个人资料](/image/个人资料.png)

- 聊天页面

![聊天页面](/image/聊天.png
)



核心代码:
-----
1. 注册使用邮箱验证码
```diff
//邮件对象
public class EmailModel {
	private String title;//标题
	private String receiverEmail;//接收人邮箱
	private String text;//发送内容
	public static String register="欢迎来到微博客\n您的注册为:";
	public static String registerTitle="注册验证码";
	public static String findPwdTitle="找回密码：";
	public static String findPwdText="找回密码验证码：";
}
// 邮件发送
public class EmailUtils {
	private final static String authorizationCode="khdotvxxxxdba"; 	//授权码-自己设置
	private final static String senderEmail ="1022287044@qq.com";	//发送人邮箱--测试邮箱
	//发送邮件方法
	public static int sendEmail(EmailModel emailModel){
     try{
         Properties props = new Properties();
         // 开启debug调试
         props.setProperty("mail.debug", "false");
         // 发送服务器需要身份验证
         props.setProperty("mail.smtp.auth", "true");
         // 设置邮件服务器主机名
         props.setProperty("mail.host", "smtp.qq.com");
         // 发送邮件协议名称
         props.setProperty("mail.transport.protocol", "smtp");
         MailSSLSocketFactory sf = new MailSSLSocketFactory();
         sf.setTrustAllHosts(true);
         props.put("mail.smtp.ssl.enable", "true");
         props.put("mail.smtp.ssl.socketFactory", sf);
         Session session = Session.getInstance(props);
         Message msg = new MimeMessage(session);
         msg.setSubject(emailModel.getTitle());//标题
         StringBuilder builder = new StringBuilder();
         builder.append(emailModel.getText());
         msg.setText(builder.toString());
         msg.setFrom(new InternetAddress(senderEmail));//发送人的邮箱地址
         Transport transport = session.getTransport();
         //发送人的邮箱地址 //你的邮箱密码或者授权码
         transport.connect("smtp.qq.com", senderEmail, authorizationCode);
         transport.sendMessage(msg, new Address[] { new InternetAddress(emailModel.getReceiverEmail()) });// 接收人的邮箱地址
         transport.close();
         return 1;
     }catch (Exception e){
        e.printStackTrace();
        return 0;
     }
	 }
}

//注册及发送邮箱验证码
public void doGet(HttpServletRequest request, HttpServletResponse response)  {
    try{
        PrintWriter printWriter=response.getWriter();
        HttpSession session=request.getSession();
        String action = request.getParameter("action");
        String userName=request.getParameter("userName");
        String password=request.getParameter("password");
         if("sendEmail".equals(action)){			//发送验证码
            String receiverEmail=request.getParameter("receiverEmail");
            String type=request.getParameter("type");
            String authCode=RandomUtil.randomNumbers(4);
            EmailModel emailModel=new EmailModel();
            if("register".equalsIgnoreCase(type)){ //注册
                User search=new User();
                search.setUserName(receiverEmail);
                search=userService.getUser(search);
                if(null!=search){
                    printWriter.println("2");
                    return;
                }else{
                    emailModel.setTitle(EmailModel.registerTitle);
                    emailModel.setText(EmailModel.register+authCode);
                }
            }else{								//找回密码
                emailModel.setTitle(EmailModel.findPwdTitle);
                emailModel.setText(EmailModel.findPwdText+authCode);
            }
            System.out.println("邮箱验证码："+authCode);
            emailModel.setReceiverEmail(receiverEmail);
            int result= EmailUtils.sendEmail(emailModel);
            session.setAttribute(receiverEmail+"#EmailCode",authCode);//注册验证码
            printWriter.println(result);

        }else if("doRegister".equals(action)){					//注册验证
            String authCode=request.getParameter("authCode");//验证码
            String sessionCode=(String)session.getAttribute(userName+"#EmailCode");//注册验证码
            if(null==sessionCode){
                printWriter.println("3");					//验证码为空，请先获取邮箱验证码
            }else if(!sessionCode.equals(authCode)){
                printWriter.println("2");					//验证码错误
            }else{
                User user=new User();
                user.setUserName(userName);
                user.setNickName(userName);
                user.setPassword(password);
                int result=userService.addUser(user);
                printWriter.println(result);				//验证成功
            }
        }
    }catch (Exception e){
        e.printStackTrace();
        return ;
    }
}
```

2. 上传头像
```diff
protected void doGet(HttpServletRequest request, HttpServletResponse response) {
    try {
        // 配置上传参数
        PrintWriter printWriter = response.getWriter();
        String fileName = "";
        DiskFileItemFactory factory = new DiskFileItemFactory();
        ServletFileUpload upload = new ServletFileUpload(factory);
        String userId = request.getParameter("userId");
        String type = request.getParameter("type");
        if (null != userId && !"".equals(userId)) {
            List<FileItem> formItems = upload.parseRequest(request);
            for (FileItem item : formItems) {            // 迭代表单数据
                if (!item.isFormField()) {
                    String fileName2 = item.getName();    //重置乱码中文
                    fileName = RandomUtil.randomString(5) + (fileName2.substring(fileName2.indexOf(".")));
                    String filePath = String.format("%s/%s", SystemConfig.fileUploadPath, fileName);
                    File storeFile = new File(filePath);
                    item.write(storeFile);                    // 保存文件到硬盘
                }
            }
            if (null == type || "".equalsIgnoreCase(type)) {
                String userUpload = SessionUtils.getUserUpload(Integer.valueOf(userId), request);
                if (null != userUpload && !"".equalsIgnoreCase(userUpload)) {
                    userUpload = userUpload + fileName + ",";
                } else {
                    userUpload = fileName + ",";
                }
                SessionUtils.setUserUpload(Integer.valueOf(userId), userUpload, request);
                printWriter.println("0");
            } else if ("headPic".equalsIgnoreCase(type)) {//头像
                SessionUtils.setUserHeadPic(Integer.valueOf(userId), fileName, request);
                printWriter.println("0");
            }
        } else {
            printWriter.println("0");    //用户信息未找到
        }
        printWriter.close();
    } catch (Exception ex) {
        ex.printStackTrace();
    }
}
```
 
3. 个人聊天（发送消息及读取消息实现）

```
//消息采用读取数据库模式实现，并非socket或者其他通讯主件
public void doGet(HttpServletRequest request, HttpServletResponse response)  {
    HttpSession session=request.getSession();
    PrintWriter printWriter=response.getWriter();
    String action = request.getParameter("action");
    User user= SessionUtils.getUser(request);
    if ("sendMsg".equals(action)) {		//发送消息
        String message=request.getParameter("message");
        String receiveId=request.getParameter("receiveId");
        Chat chat=new Chat();
        chat.setUserId(user.getId());
        chat.setUserName(user.getNickName());
        chat.setMessage(message);
        chat.setStatus(0);
        chat.setReceiveId(Integer.valueOf(receiveId));
        chat.setCreateTime(DateUtil.formatDateTime(new Date()));
        Posts posts=new Posts();
        posts.setReleaseId(user.getId());
        chatService.addChat(chat);
        printWriter.println("0");
        session.setAttribute(user.getId()+":"+receiveId,message);
        //添加推送
        Notice notice=new Notice();
        notice.setNoticeUserId(Integer.valueOf(receiveId));
        notice.setUserId(user.getId());
        notice.setNoticeType(3);
        notice.setIsRead(0);
        Notice isExtis=noticeService.getNotice(notice);
        if(null==isExtis){
            System.out.println("添加推送");
            notice.setUserName(user.getUserName());
            notice.setPostTitle(message);
            noticeService.addNotice(notice); //添加通知
        }else{
            System.out.println("有未读消息，不添加推送");
        }
    }else if ("getMsg".equals(action)) {		//获取消息
        String sendUserId=request.getParameter("sendUserId"); //发送人id
        String receiveId=request.getParameter("receiveId"); //接收人
        Chat chat=new Chat();
        chat.setUserId(Integer.valueOf(sendUserId));
        chat.setReceiveId(Integer.valueOf(receiveId));
        chat.setStatus(0);
        List<Chat> list=chatService.getChatList(chat);
        if(list.size()>0){
            printWriter.println(JSONUtil.toJsonStr(list));
            for(Chat c:list){
                c.setStatus(1);
                chatService.updateChat(c);
            }
        }else{
            printWriter.println("");
        }
        printWriter.close();
    }
}
//前端jsp页面代码
<div class="qqBox">
    <div class="BoxHead">
        <div class="headImg">
            <img src="images/upload/${userInfo.headPic}" />
            <!--当前自己-->
            <input value="images/upload/${loginUser.headPic}" id="loginUserHeadPic" type="hidden"/>
            <input value="${loginUser.id}" id="sendUserId" type="hidden"/>

            <!--对方好友-->
            <input value="images/upload/${userInfo.headPic}" id="userInfoHeadPic" type="hidden"/>
            <input value="${userInfo.id}" id="receiveId" type="hidden"/>
        </div>
        <div class="internetName">${userInfo.userName}</div>
    </div>
    <div class="context">
        <div class="conRight">
            <div class="Righthead">
                <div class="headName">${userInfo.nickName}</div>
                <div class="headConfig">
                    <ul>
                        <li><img src="images/chat/20170926103645_06.jpg"/></li>
                        <li><img src="images/chat/20170926103645_08.jpg"/></li>
                        <li><img src="images/chat/20170926103645_10.jpg"/></li>
                        <li><img src="images/chat/20170926103645_12.jpg"/></li>
                    </ul>
                </div>
            </div>
            <div class="RightCont">
                <ul class="newsList">

                </ul>
            </div>
            <div class="RightFoot">
                <div class="footTop">
                    <ul>
                        <li><img src="images/chat/20170926103645_31.jpg"/></li>
                        <li class="ExP"><img src="images/chat/20170926103645_33.jpg"/></li>
                        <li><img src="images/chat/20170926103645_35.jpg"/></li>
                        <li><img src="images/chat/20170926103645_37.jpg"/></li>
                        <li><img src="images/chat/20170926103645_39.jpg"/></li>
                        <li><img src="images/chat/20170926103645_41.jpg" alt="" /></li>
                        <li><img src="images/chat/20170926103645_43.jpg"/></li>
                        <li><img src="images/chat/20170926103645_45.jpg"/></li>
                    </ul>
                </div>
                <div class="inputBox">
                    <textarea id="dope" style="width: 99%;height: 75px; border: none;outline: none;" name="" rows="" cols=""></textarea>
                    <button class="sendBtn">发送(s)</button>
                </div>
            </div>
        </div>
    </div>
</div>

```

4. 关注 粉丝 喜欢统计
```diff
//每一项根据用户id单独统计在组装数据
public void getLikeFansNul(Integer userId,HttpServletRequest request){
    String fansSql="select count(id) from t_likeuser where likeUserId="+userId;
    int fansNum= JDBCUtils.getCount(fansSql);
    String likeSql="select count(id) from t_likeuser where userid="+userId;
    int likeNum= JDBCUtils.getCount(likeSql);
    String collectSql="select count(id)  from t_collection where userid="+userId;
    int collectNum= JDBCUtils.getCount(collectSql);
    CountsVo countsVo=new CountsVo(fansNum,likeNum,collectNum);
    SessionUtils.setCountsVo(countsVo,request);
}

``` 

项目注意事项
----
1: 收藏和关注才会有通知 <br>
2：收藏自己的推文没有通知<br>
3：聊天消息推送如果有一条当前人发送的消息未读时，只保存一条未读消息<br>
4：消息采用读取数据库模式实现，并非socket或者其他通讯主件,后期会单独起一个项目完成socket版本及其他三方开源的通信主件版本，敬请关注及更新<br>


