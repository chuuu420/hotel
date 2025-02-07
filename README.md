//酒店
// 24爪哇后端第一轮考核
#include <stdio.h>
#include <stdlib.h>
#include <math.h>
#include <time.h>  
#include <string.h>


//管理员需要先添加房间和价格，否则用户不能预订房间和查找房间
//修改用户余额那部分函数完成的不好，有错误
//这一串代码在编写过程中有借助AI工具进行完善


//typedef关键字用于为已有的数据类型创建一个新的名称 struct{}定义了一个匿名结构体，后面的user是新名称
typedef struct{
	char username[50];
	char mima[50];
	int shenfen; //1是管理员 0是顾客
	int vip; //1是会员 0是普通顾客
	float money;//顾客余额
}user;

typedef struct{
	int roomID;//房号
	char type[50];//房间类型
	float price;//房间价格
	int isbook;//是否被预定 1：已经被预定 0：没有被预定
}room;

#define max_rooms 100
#define max_users 100
user users[max_users];
room rooms[max_rooms];
int usercount=0;//用户的数量
int roomcount=0;//房间的数量

//加载用户的信息
void loadusers(){
	FILE *file=fopen("users.dat","rb");
	//使用fopen函数打开users.dat的文件，rb表示以二进制只读的模式打开文件，如果文件打开成功会返回一个指向FILE类型的指针赋值给file，失败的话file就是NULL
	if(file){
		//读取的是用户的数量
		fread(&usercount,sizeof(int),1,file);
		//fread函数是从文件中读取数据，fread(指向要存储读取数据的内存地址，每个数据项的大小，要读取的数据项数量，指向要读取的文件的指针)
		//读取用户的数据
		fread(users,sizeof(user),usercount,file);
		fclose(file);
	}
}

//保存用户信息
void saveusers(){
	FILE *file=fopen("users.dat","wb");//打开文件，如果文件不存在将创建，文件存在将清空
	if(file){
		fwrite(&usercount,sizeof(int),1,file);//写入用户的数量
		fwrite(users,sizeof(user),usercount,file);//
		fclose(file);
	}
}

//加载房间数据
void loadrooms(){
	FILE *file=fopen("rooms.dat","rb");
	if(file){
		fread(&roomcount,sizeof(int),1,file);
		fread(rooms,sizeof(room),roomcount,file);
		fclose(file);
	}
}

//保存房间数据
void saverooms(){
	FILE *file=fopen("rooms.dat","wb");
	if(file){
		fwrite(&roomcount,sizeof(int),1,file);
		fwrite(rooms,sizeof(room),roomcount,file);
		fclose(file);
	}
}

//添加用户
void addusers(){
	if(usercount>=max_users){
		printf("用户数量达到上限！");
		return;
	}
	user newuser;
	printf("请输入用户名：");
	scanf("%49s",newuser.username);
	printf("请输入密码：");
	scanf("%49s",newuser.mima);
	
	int role;
	printf("请选择用户角色（1：管理员 0：顾客）：");
	scanf("%d",&role);
	newuser.shenfen=role;
	if(role==0){
		int leixing;
		printf("请选择顾客类型（1：会员 0：普通客户）：");
		scanf("%d",&leixing);
		newuser.vip=leixing;
		newuser.money=0.0;//初始大家的余额
	}else{
		newuser.vip=0;//管理员没有会员
		newuser.money=0.0;
	}
	
	users[usercount++]=newuser;//加入新的信息
	saveusers();
	printf("用户添加成功！\n");
}

//删除用户
void deleteusers(){
	char username[50];
	printf("请输入要删除的名字：");
	scanf("%s",username);
	for(int i=0;i<usercount;i++){
		if(strcmp(users[i].username,username)==0){
			for(int j=i;j<usercount-1;j++){
				users[j]=users[j+1];
			}
			usercount--;
			saveusers();
			printf("用户删除成功！\n");
			return;
		}
	}
	printf("查无此用户！\n");
}

//修改用户信息
void xiugaiusers(){
	char username[50];
	printf("请输入要修改的用户名字：");
	scanf("%49s",username);
	for(int i=0;i<usercount;i++){
		if(strcmp(users[i].username,username)==0){
			printf("请输入新密码：");
			scanf("%49s",users[i].mima);
			if(users[i].shenfen==0){
				int leixing;
				printf("请选择新的顾客类型（1：会员，0：普通顾客)：");
				scanf("%d",&leixing);
				users[i].vip=leixing;
			}
			saveusers();
			printf("用户信息修改成功\n");
			return;
		}
	}
	printf("查无此用户！\n");
}

//查询用户信息
void informationusers(){
	char username[50];
	printf("请输入要查询的用户名字：");
	scanf("%49s",username);
	
	for(int i=0;i<usercount;i++){
		if(strcmp(users[i].username,username)==0){
			printf("用户名:%s\n",users[i].username);
			printf("角色：%s\n",users[i].shenfen?"管理员":"顾客");
			if(users[i].shenfen==0){
				printf("，顾客类型：%s\n",users[i].vip?"会员顾客":"普通顾客");
				printf("余额：%.2f\n",users[i].money);
			}
			return;
		}
	}
	printf("未找到该用户！\n");
}

// 显示所有用户
void listusers(){
	for(int i=0;i<usercount;i++){
		printf("用户名：%s,角色：%s",users[i].username,users[i].shenfen?"管理员":"顾客");
		if(users[i].shenfen==0){
			printf("，顾客类型：%s,余额：%.2f",users[i].vip?"会员顾客":"普通顾客",users[i].money);
		}
		printf("\n");
	}
}

//添加房间
void addrooms(){
	if(roomcount>=max_rooms){
		printf("房间数量已达上限！\n");
		return;
	}
room newroom;
newroom.roomID=roomcount+1;
printf("请输入房间类型：");
scanf("%s",newroom.type);
printf("请输入房间价格：");
scanf("%f",&newroom.price);
newroom.isbook=0;//一开始是空闲状态没有被预定
rooms[roomcount++]=newroom;
saverooms();
printf("房间添加成功！\n");
}

//删除房间
void deleterooms(){
	int roomid;
	printf("请输入你要删除的房间号：");
	scanf("%d",&roomid);
	for(int i=0;i<roomcount;i++){
		if(rooms[i].roomID==roomid){
			for(int j=i;j<roomcount-1;j++){
				rooms[j]=rooms[j+1];
			}
			roomcount--;
			saverooms();
			printf("房间删除成功！\n");
			return;
		}
	}
	printf("没有此房间！\n");
}

//查询房间的信息
void checkrooms(){
	for(int i=0;i<roomcount;i++){
		printf("房间号：%d,类型：%s,价格：%.2f，状态:%s\n",rooms[i].roomID,rooms[i].type,rooms[i].price,rooms[i].isbook?"有人":"空闲");
	}
}

//预订房间
void bookrooms(int userID){
	int roomid;
	printf("请输入要预定的房号：");
	scanf("%d",&roomid);
	for(int i=0;i<roomcount;i++){
		if(rooms[i].roomID==roomid){
			if(rooms[i].isbook){
				printf("抱歉！该房间已经被预定，请另外选择合适的房间!\n");
				return;
			}
			if(users[userID].money<rooms[i].price){
				printf("余额不足，请充值后再进行预订或选择另外合适的房间！\n");
				return;
			}
			rooms[i].isbook=1;
			users[userID].money-=rooms[i].price;
			saverooms();
			saveusers();
			printf("房间预订成功！\n");
			return;
		}
	}
	printf("未找到该房间！\n");
}

//修改用户余额
void xiugaimoney(){
	char username[50];
	float newmoney;
	printf("请输入要修改余额的用户名字：");
	scanf("%s",username);
	for(int i=0;i<usercount;i++){
		if(strcmp(users[i].username,username)==0){
			if(users[i].shenfen==1){
				printf("请输入新的余额：");
				scanf("%f",&newmoney);
				if(newmoney>=0){
					users[i].money=newmoney;
					saveusers();
					printf("用户余额修改成功！\n当前余额为%.2f\n",users[i].money);
				}else{
					printf("余额不能是负数，重新操作\n");
				}
			}
		}else{
			printf("用户不能擅自修改余额！");
		}
		return;
	}
}

//管理员管理用户的目录
//增加：addusers() 删除：deleteusers() 修改：xiugaiusers() 查询：informationusers() 全部用户信息：listusers() 
void yonghu(){
	int choice;
	do{
		printf("\n********用户管理目录********\n");
		printf("1.添加用户\n");
		printf("2.删除用户\n");
		printf("3.修改用户\n");
		printf("4.查询用户\n");
		printf("5.列出全部用户信息\n");
		printf("6.修改用户余额\n");
		printf("0.返回主菜单\n");
		scanf("%d",&choice);
		switch(choice){
			case 1:addusers();break;
			case 2:deleteusers();break;
			case 3:xiugaiusers();break;
			case 4:informationusers();break;
			case 5:listusers();break;
			case 6:xiugaimoney();break;
			case 0:printf("返回主菜单\n");break;
			default:printf("无效操作，请重试！\n");
		}
	}while(choice!=0);
}

//管理员管理房间目录
//添加：addrooms() 删除：deleterooms()  查询：checkrooms() 
void fangjian(){
	int choice;
	do{
		printf("\n********房间管理目录********\n");
		printf("1.添加房间\n");
		printf("2.删除房间\n");
		printf("3.查询房间\n");
		printf("4.预定房间(管理员代订)\n");
		printf("0.返回主菜单\n");
		printf("请输入操作数：");
		scanf("%d",&choice);
		char username[50];
		switch(choice){
			case 1:addrooms();break;
			case 2:deleterooms();break;
			case 3:checkrooms();break;
			case 4:{
			printf("请输入要代订房间的顾客用户名：");
			scanf("%s",username);
			int userID=-1;
			for(int i=0;i<usercount;i++){
				if(strcmp(users[i].username,username)==0){
					userID=i;
					break;
				}
			}
			if(userID==-1){
				printf("无此顾客，无法预订\n");
			}else{
				bookrooms(userID);
			}
			break;}
			case 0:printf("返回主菜单\n");break;
			default:printf("无效操作，请重试！\n");
		}
	}while(choice!=0);
}

//顾客目录
//查询房间 checkrooms() 预订房间  bookrooms 
void guke(int userID){
	int choice;
	do{
		printf("\n********顾客目录********\n");
		printf("1.查询房间信息\n");
		printf("2.预订房间\n");
		printf("3.查询本账户余额\n");
		printf("0.返回主菜单\n");
		printf("请输入操作数：");
		scanf("%d",&choice);
	switch(choice){
		case 1:checkrooms();break;
		case 2:bookrooms(userID);break;
		case 3:printf("当前余额：%.2f\n",users[userID].money);break;
		case 0:printf("返回主菜单\n");break;
		default:printf("无效操作，请重试\n");
	}
}while(choice!=0);
}

void guanliyuan(){
	int choice;
	do{
		printf("\n********管理员目录******\n");
		printf("1.房间目录\n");
		printf("2.客人目录\n");
		printf("0.返回主菜单\n");
		printf("请输入操作数：");
		scanf("%d",&choice);
		switch(choice){
		case 1:fangjian();break;
		case 2:yonghu();break;
		case 0:printf("f返回主菜单\n");break;
		default:printf("无效操作 请重试！\n");
	}
}while(choice!=0);
}

//登陆
int login(){
	char username[50],mima[50];
	printf("请输入用户名：");
	scanf("%s",username);
	printf("请输入密码：");
	scanf("%s",mima);
	for(int i=0;i<usercount;i++){
		if(strcmp(users[i].username,username)==0&&strcmp(users[i].mima,mima)==0){
			printf("登陆成功！\n");
			return i;//返回用户的索引
		}
	}
	printf("用户名或密码错误！\n");
	return -1;
}

int main(){
	loadusers();
	loadrooms();
	int mainchoice;
	do{
		printf("\n********主菜单********\n");
		printf("1.顾客登陆\n");
		printf("2.管理员登陆\n");
		printf("0.退出\n");
		printf("请输入操作数：");
		scanf("%d",&mainchoice);
		int userid;
		switch(mainchoice){
			case 1:
			userid=login();
			if(userid !=-1&&users[userid].shenfen==0){
			guke(userid);	
			}
			break;
			case 2:guanliyuan();break;
			case 0:printf("退出系统\n");break;
			default:printf("无效操作请重试！\n");
		}
	}while(mainchoice!=0);
	return 0;
}






