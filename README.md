# sunqingqing
2016011541

Linux作业
=========

首先, chmod命令有2个格式<br>
1）  chmod 777 test.c//数字格式<br>
2) chmod a+x, u+rw test.c //字符格式<br>
因为是2个格式， 所以我们用分别处理它们<br>

首先是"数字格式"
----------------
"数字(即777)"是需要写入的权限<br>
因为"数字"是中命令行获取的,而从命令行获取的信息都是字符串的形式存在的，所以我们需要将数字(字符串类型)，转换成整型<br>
所以需要用到atoi()函数(atoi()函数功能: 将字符串转换成整型)，执行了mode = atoi("数字")后，mode现在是一个整型了<br>
因为在文件的权限中有3中权限r,(可读) w(可写), x(可执行), 分别对应r = 4, w = 2, x = 1. <br>
即如果满权限的话就是r+w+x = 7, 无权限的话就是0, 因为文件有3个组，文件所有者，组所属, 其他人。所以最大的权限为777, 最小为000.<br>
因为atoi()函数返回的类型是一个整型数， 即一个十进制<br>
而chmod(const char *path, mode_t mode)中mode需要的数是一个八进制的数<br>
所以我们现在要将mode转换成一个八进制<br>
` ``c
mode_u = mode / 100;					//文件所有者权限
mode_g = (mode - (mode_u*100))/10;			//所属组权限
mode_o = mode - (mode_u*100) - (mode_g*10)		//其他人权限
mode = (mode_u * 8 * 8) + (mode_g * 8) + mode _o	//转换成八进制
` ``

chmod a+x, u+rw test.c//字符格式
------------------------------
首先在分析+，-， =的时候， 先说明一下宏<br>
宏宏对应的八进制含义<br>
S_IRWXU  00700 权限，代表该文件所有者具有可读、可写及可执行的权限。<br>
S_IRUSR 或S_IREAD，      00400权限， 代表该文件所有者具有可读取的权限。<br>
S_IWUSR 或S_IWRITE，   00200 权限， 代表该文件所有者具有可写入的权限。<br>
S_IXUSR 或S_IEXEC，      00100 权限， 代表该文件所有者具有可执行的权限。<br>
S_IRWXG 00070权限，代表该文件用户组具有可读、可写及可执行的权限。<br>
S_IRGRP 00040 权限，代表该文件用户组具有可读的权限。<br>
S_IWGRP 00020权限，代表该文件用户组具有可写入的权限。<br>
S_IXGRP 00010 权限，代表该文件用户组具有可执行的权限。<br>
S_IRWXO 00007权限，代表其他用户具有可读、可写及可执行的权限。<br>
S_IROTH 00004 权限，代表其他用户具有可读的权限.<br>
S_IWOTH 00002权限，代表其他用户具有可写入的权限。<br>
S_IXOTH 00001 权限，代表其他用户具有可执行的权限。<br>

首先， 在chmod命令中 +代表添加权限， -代表删除权限, =代表设置权限.<br>
前2个权限很好理解， 但是第3个是什么意思呢， 是这样的<br>
例如 我们有一个文件test.c 它的权限是 rwxrw-rw-<br>
现在我们用chmod u=x test.c 后， 这个文件的权限就变成了--xrw-rw, u=x代表中文件所有者的这个组中， 除了x(可执行)其他权限都被覆盖<br>
对字符格式的分析， 更多的是分析+,--,=这些操作<br>
首先我们要先构造一个架构<br>
即如果是+, 我们应该这么实现， 如果是-, 怎么实现等<br>
` ``c
+’最终框架
		case '+':
			if(u)
			{
				if(r)
					*mode |= S_IRUSR;
				if(w)
					*mode |= S_IWUSR;
				if(x)
					*mode |= S_IXUSR;
			}
 
			if(g)
			{
				if(r)
					*mode |= S_IRGRP;
				if(w)
					*mode |= S_IWGRP;
				if(x)
					*mode |= S_IXGRP;
			}
 
			if(o)
			{
				if(r)
					*mode |= S_IROTH;
				if(w)
					*mode |= S_IWOTH;
				if(x)
					*mode |= S_IXOTH;	
			}
			break;
  ` ``
  
  接下来分析'-'.<br>
 '-'是删除权限， 即如果权限是r-xrwxrwx的话， 那么chmod u-x后， 权限就是r--rwxrwx<br>
这里就是屏蔽要删除的权限， 在这里我们可以用与运算+反码实现这种操作<br>
原理是0&任何数=0<br>
这里的思路， 例如现在我要删除u的r权限， 而u的r权限的宏S_IXUSR的八进制是00100, 现在将它取反就变成了677 (不管前2个代表什么)， <br>
在没有chmod u-x之前 权限是r-xrwxrwx 即577<br>
现在我们用577&677<br>

101 111 111<br>
& 110 111 111<br>
____________________
100 111 111  = 477<br>

权限477 翻译后， 权限为r--rwxrwx 很明显u的x权限屏蔽了<br>

` ``c
‘-’最终框架
		case '-':
			if(u)
			{
				if(r)
					*mode &= ~S_IRUSR;
				if(w)
					*mode &= ~S_IWUSR;
				if(x)
					*mode &= ~S_IXUSR;
			}
 
			if(g)
			{
				if(r)
					*mode &= ~S_IRGRP;
				if(w)
					*mode &= ~S_IWGRP;
				if(x)
					*mode &= ~S_IXGRP;
			}
 
			if(o)
			{
				if(r)
					*mode &= ~S_IROTH;
				if(w)
					*mode &= ~S_IWOTH;
				if(x)
					*mode &= ~S_IXOTH;	
			}
			break;
` ``
最后分析'='<br>
= 的操作时一个屏蔽权限的过程，<br> 
例如文件test.c的权限是rwxrwxrwx, 执行chmod r=x后,<br>
权限就变成了--xrwxrwx, 即在所有者的组中rw权限被屏蔽，<br> 
而组所属组和其他人组没有发生什么变化<br>
所以这里我们可以将需要覆盖权限的组的rwx全部置0，其他组不变， <br>
然后在加上需要加入的权限<br>
例如在上面的例子中， <br>
我们可以这样做<br>
` ``c
‘=’最终框架
		case '=':
			if(u)
			{
				*mode &= (S_IRGRP|S_IWGRP|S_IXGRP|S_IROTH|S_IWOTH|S_IXOTH);
				if(r)
					*mode |= S_IRUSR;
				if(w)
					*mode |= S_IWUSR;
				if(x)
					*mode |= S_IXUSR;
			}
 
			if(g)
			{
				*mode &= (S_IRUSR|S_IWUSR|S_IXUSR|S_IROTH|S_IWOTH|S_IXOTH);
 				if(r)
					*mode |= S_IRGRP;
				if(w)
					*mode |= S_IWGRP;
				if(x)
					*mode |= S_IXGRP;
			}
			if(o)
			{
				*mode &= (S_IRGRP|S_IWGRP|S_IXGRP|S_IRUSR|S_IWUSR|S_IXUSR);
				if(r)
					*mode |= S_IROTH;
				if(w)
					*mode |= S_IWOTH;
				if(x)
					*mode |= S_IXOTH;
			}
			break;
` ``
