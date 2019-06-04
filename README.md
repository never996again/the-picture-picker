# the-picture-picker
this implement is use for create or delete rectangles in the picture with json,it is usually work for train the face detect in AI
#-*-coding:utf-8-*-
import os,sys
import numpy as np
import cv2
import json
import shutil
drawing=False
reload=False
delete=False
graylist=[]
showtxt=False
threadhold=0.15
stop=False
direct='next'
ix,iy=-1,-1
lx,ly=-1,-1
jsonpath='/home/pixel/Desktop/7000/down7.json'#获取json文本路径
with open(jsonpath,'r')as f:
    piclist=json.load(f)
print '＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊使用说明＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊'
print '1.按Ｖ建切换模式，抹灰/画框'
print '2.按空格键到下一张'
print '3.按ｂ键到上一张'
print '4.按ｄ键删除此张，再按空格键下一张'
print '5.按ｑ键暂停'
i=int(input('请输入编号：'))
keylist=list(piclist.keys())
keylist.sort()#给路径名排序，防止每次得到的路径名不一样
delmenu=set()
rootpath='/home/pixel/Desktop/7000/down7/'
def draw(event,x,y,flags,param):
    global lx,ly,ix,iy,drawing,gray,showimg,img,piclist,keylist,i,valuelist,delete,reload,grayimg,oriimg,tmpimg,imgpath,graylist
    if event==cv2.EVENT_LBUTTONDOWN:#按下鼠标左键，获取初时坐标点，置画框删除参数为真
        drawing=True
        ix,iy=x,y
        delete=True
    elif event==cv2.EVENT_RBUTTONDOWN:#按下鼠标右键，若当前坐标点在某个灰色或绿色框范围内删除该框，且若为绿色框写回json文本，因为修改了json所以重新加载showimg
        if delete==True:
            if gray==True:
                for it in range(len(graylist)):
                    if (ix>graylist[it][0] and ix<graylist[it][2]) and (iy<graylist[it][3] and iy>graylist[it][1]):
                        del graylist[it]
                        break
            else:
                if len(valuelist)>0:
                    for ci in range(len(valuelist)):
                        if (ix > int(valuelist[ci][0]) and ix < int(valuelist[ci][2])) and (
                                int(iy > valuelist[ci][1]) and int(iy < valuelist[ci][3])):
                            with open(jsonpath, 'w')as fw:
                                del valuelist[ci]
                                # del valuelist[1][ci]
                                json.dump(piclist, fw)
                                break
        reload=True
    elif event==cv2.EVENT_RBUTTONUP:#重新加载json
            reload=True
    elif event==cv2.EVENT_MOUSEMOVE:#鼠标移动时动态显示画的框
        if drawing==True:
            # showimg=img.copy()
            if (x-ix)>3 and (y-iy)>3:
                if gray==True:
                    showimg=img.copy()#每次画框都是在原图的基础上画框，不加这条画的框是连续的
                    cv2.rectangle(showimg,(ix,iy),(x,y),(255,255,0),-1)
                else:
                    showimg=img.copy()
                    cv2.rectangle(showimg,(ix,iy),(x,y),(0,255,0))
    elif event==cv2.EVENT_LBUTTONUP:#画框结束，写入灰色框或写回json文本
        drawing=False
        # delete=False
        if (x-ix)>5 and (y-iy)>5:
            if gray==True:
                a=[ix,iy,x,y]
                graylist.append(a)
            else:
                with open(jsonpath,'w')as fw:
                    clist=[ix,iy,x,y]
                    valuelist.append(clist)
                    json.dump(piclist,fw)
        reload=True
while i<len(keylist):
    graylist=[]
    gray=False
    imgpath=os.path.basename(keylist[i])
    if os.path.exists(rootpath+imgpath):
        notion='按Ｖ键切换画框/抹灰'
        cv2.namedWindow(imgpath, cv2.WINDOW_NORMAL)
        cv2.setMouseCallback(imgpath, draw)
        img = cv2.imread(rootpath+imgpath)#img为原图
        # height,weight,channel = img.shape
        height,weight,channel=img.shape
        oriimg=img.copy()#oriimg开始时为原图
        showimg=img.copy()#开始时显示原图
        valuelist=piclist[keylist[i]]
        stop=False
        print('当前位置为：'+str(i))
        # print(valuelist)
        if len(valuelist)>0:#画出json文本中原图的框
            for ji in range(len(valuelist)):
                cx1, cy1, cx2, cy2 = valuelist[ji][0], valuelist[ji][1], valuelist[ji][2], valuelist[ji][3]
                # if valuelist[1][j]>=threadhold:
                # x1,y1,x2,y2=int(valuelist[1][j][0]),int(valuelist[1][j][1]),int(valuelist[1][j][2]),int(valuelist[1][j][3])
                cv2.rectangle(showimg,(cx1,cy1),(cx2,cy2),(0,255,0))
                    # cv2.putText(showimg, "%.3f" % valuelist[1][j], (valuelist[0][j][0], valuelist[0][j][3]), cv2.FONT_ITALIC, 0.6,
                    #             (0, 0, 255), 1)
        img=showimg.copy()#显示图片的备份暂时更新为带框图
        while True:
            #如果reload就重新加载图片，重新读入json文本中的框
            if reload==True:
                showimg=cv2.imread(rootpath+imgpath)
                if len(valuelist)>0:
                    for j in range(len(valuelist)):
                        # if valuelist[1][j] >= threadhold:
                            cv2.rectangle(showimg, (valuelist[j][0], valuelist[j][1]),
                                          (valuelist[j][2],valuelist[j][3]), (0, 255, 0))
                            # cv2.putText(showimg, "%.3f" % valuelist[1][j], (valuelist[0][j][0], valuelist[0][j][3]),
                            #             cv2.FONT_ITALIC, 0.6,
                            #             (0, 0, 255), 1)
                for l in graylist:
                    cv2.rectangle(showimg,(l[0],l[1]),(l[2],l[3]),(255,255,0),-1)
                img=showimg.copy()
                # print valuelist
                # print graylist
                reload=False
            # cv2.imshow(keylist[i],showimg)
            cv2.moveWindow(imgpath,1,1)
            cv2.resizeWindow(imgpath,1800,1000)
            cv2.imshow(imgpath,showimg)
            k=cv2.waitKey(5)
            # for qw in graylist:
            #     cv2.rectangle(oriimg, (qw[0], qw[1]), (qw[2], qw[3]), (255, 255, 0), -1)
            # cv2.imwrite(rootpath + imgpath, oriimg)
            # cv2.imwrite(rootpath+imgpath,oriimg)
            if k==ord('v'):
                gray=not gray#切换模式
            if k == ord(" "):#当切换到上一张/下一张时或暂停时将抹灰后的图片修改回原目录
                for qw in graylist:
                    cv2.rectangle(oriimg,(qw[0],qw[1]),(qw[2],qw[3]),(255,255,0),-1)
                cv2.imwrite(rootpath+imgpath,oriimg)
                i+=1
                direct='next'
                cv2.destroyAllWindows()
                break
            if k==ord('d'):
                delmenu.add(imgpath)
                img5=unicode('/home/pixel/Desktop/7000/down7/'+imgpath)
                shutil.copy(img5,'/home/pixel/Desktop/7000/double/')
                doulist=os.listdir('/home/pixel/Desktop/7000/double/')
                print '重复框图片书为：'
                print len(doulist)
                # i+=1
                break
                cv2.destroyAllWindows()
            if k==ord('q'):
                for qw in graylist:
                    cv2.rectangle(oriimg, (qw[0], qw[1]), (qw[2], qw[3]), (255, 255, 0), -1)
                cv2.imwrite(rootpath + imgpath, oriimg)
                stop=True
                cv2.destroyAllWindows()
                break
            if k==ord('b'):
                for qw in graylist:
                    cv2.rectangle(oriimg,(qw[0],qw[1]),(qw[2],qw[3]),(255,255,0),-1)
                cv2.imwrite(rootpath+imgpath,oriimg)
                cv2.destroyAllWindows()
                i-=1
                direct='last'
                break
        if stop:break
    else:
        if direct=='next':
            i+=1
        else:i-=1
if stop and stop==True:
    print("暂停位置为"+str(i))
else:
    print('所有图片都挑完了')
for qw in delmenu:
    os.remove('/home/pixel/Desktop/7000/down7/'+qw)
