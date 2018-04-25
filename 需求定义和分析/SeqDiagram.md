# 系统消息顺序图

## Notice

注意到我把一些控制功能交给了Page，因为这些功能太简单了，不用过于担心，MVVM中的Page和视图控制器通过同一个类：**ViewModel** 进行建模即可。因此我们组在设计阶段的类的数量将会减少。

简单起见。软件的任何异常都在开发阶段进行处理，不在设计中设计这部分内容。因过于繁琐。

## Reference Class List



Class Name|Description
:-|:-
User|存储各种用户信息
SoundSource|虚拟声源
VirtualSoundSpace|虚拟声音空间抽象类
MapModeVirtualSoundSpace|地图模式虚拟声音空间
ConcertModeSoundSpace|音乐会模式虚拟声音空间
LocalSoundSpaceLibrary|本地VSS管理
OnlineSoundSpaceLibrary|在线VSS管理库
Comment|评论存储类
Like|赞存储类
Sensor|传感器接口
SoundFragment|声音片段
-|-
RegisterPage|注册界面
LoginPage|登陆界面
LocalVSSManagerPage|本地VSS管理界面
OnlineVSSViewerPage|在线VSS浏览界面
VSSViewingPage|进入VSS进行游览
CreateVSSPage|VSS创建入口界面
CreateMapVSSPage*|MapVSS创建
CreateConcertVSSPage*|ConcertVSS创建
SensorPage|传感器矫正页面
PreviewLocalMapPage|-
PreviewOnlineMapPage|-
PreviewOnlineConcertPage|-
PreviewLocalMapPage|-
UploadLocalVSSDialog|-
-|-
UserInfoControl|用户信息管理控制器
PreviewLocalConcertControl|-
PreviewOnlineConcertControl|-
PreviewLocalMapControl|-
PreviewOnlineMapControl|-
MapVSSCreateControl*|MapVSS创建控制
ConcertVSSCreateControl*|ConcertVSS创建控制
LocalVSSLibraryControl|本地VSS管理界面控制器
OnlineVSSLibraryControl|在线VSS浏览界面控制器
SensorControl|传感器校准控制器
ConcertVSSPlayControl*|CVSS播放控制器
MapVSSPlayControl*|MVSS播放控制器
GVRAudioEngine*|GVR虚拟声音播放控制器


## 1.Register
![SeqDiagram](./Register.svg)

```PlantUML
@startuml Register
hide footbox
skinparam sequenceParticipant underline

actor User as user
participant ":RegisterPage" as boundary
participant ":UserInfoControl" as control
participant ":User" as userdata
participant ":LocalVSSManagerPage" as nextpage

note right of control: The UserInfoControl is activated since the start of the entire program.

user -> boundary: enterRegisterData
boundary -> control: sendRegisterData
control -> control: checkRegisterData
loop wrong input
    control --> boundary: registerDataWrong
    user -> boundary: enterRegisterData
    boundary -> control: sendRegisterData
    control -> control: checkRegisterData
end
control --> boundary: registerSuccess
create userdata
control -> userdata: <<create>>
control -> control: setCurrentUser
control -> boundary: <<destroy>>
destroy boundary
create nextpage
control -> nextpage: <<create>>
destroy control
@enduml
```

## 2.Login

![SeqDiagram](./Login.svg)

```PlantUML
@startuml Login
hide footbox
skinparam sequenceParticipant underline

actor User as user
participant ":LoginPage" as boundary
participant ":UserInfoControl" as control
participant ":User" as userdata
participant ":LocalVSSManagerPage" as nextpage

user -> boundary: enterLoginData
boundary -> control: sendLoginData
control -> control: checkLoginData
loop wrong password or username
    control --> boundary: loginDataWrong
    user -> boundary: enterLoginData
    boundary -> control: sendLoginData
    control -> control: checkLoginData
end
control --> boundary: LoginSuccess
create userdata
control -> userdata: <<create>>
control -> control: setCurrentUser
alt remember me selected
    control -> control: setRememberUser
end
control -> boundary: <<destroy>>
destroy boundary
create nextpage
control -> nextpage: <<create>>
destroy control
@enduml
```
## 3.ManageLocalVSSLibrary

![SeqDiagram](./ManageLocalVSSLibrary.svg)

```PlantUML
@startuml ManageLocalVSSLibrary
hide footbox
skinparam sequenceParticipant underline

actor User as user
participant ":LocalVSSManagerPage" as boundary
participant ":LocalVSSLibraryControl" as control
participant ":LocalSoundSpaceLibrary" as library

create control
boundary -> control: <<create>>
control -> library: getLocalVSSList(): List
library -> control: listOfLocalVSS: List<VirtualSoundSpace>
control -> boundary: displayVSSList(listOfLocalVSS)


alt CreateVSS
    user -> boundary: createVSS
    ...described in PlayVSS...
else RenameVSS
    user -> boundary: renameVSS
    ...described in RenameVSS...
else DeleteVSS
    user -> boundary: deleteVSS
    ...described in DeleteVSS...
else UploadVSS
    user -> boundary: uploadVSS
    ...described in UploadVSS...
else PreviewVSS
    user -> boundary: previewVSS
    ...described in PreviewVSS...
end
@enduml
```

## 4.CreateVSS

![SeqDiagram](./CreateVSS.svg)

```PlantUML
@startuml CreateVSS
hide footbox
skinparam sequenceParticipant underline

actor User as user
participant ":LocalVSSManagerPage" as manage
participant ":CreateVSSPage" as page
participant ":CreateMapVSSPage" as map
participant ":CreateConcertVSSPage" as concert

user -> manage: createVSS
create page
manage -> page: <<create>>
alt create Map Mode VSS
    user -> page: createMapModeVSS
    create map
    page -> map: <<create>>
    ...Go to CreateMapVSS...
else create Concert Mode VSS
    user -> page: createConcertModeVSS
    create concert
    page -> concert: <<create>>
    ...Go to CreateConcertModeVSS...
end

@enduml
```

## 5.CreateMapModeVSS

![SeqDiagram](./CreateMapModeVSS.svg)

```PlantUML
@startuml CreateMapModeVSS
hide footbox
skinparam sequenceParticipant underline


actor User as user
participant ":MapVSSCreatePage" as createboundary
participant ":MapVSSCreateControl" as createcontrol
participant ":LocalSoundSpaceLibrary" as library
participant ":Sensor" as sensor
collections "soundSpaces:VirtualSoundSpace" as vsses
participant "newSoundSpace:MapModeVirtualSoundSpace" as newspace


[-> createboundary: <<create>>
create createcontrol
createboundary -> createcontrol: <<create>>
create newspace
createcontrol -> newspace: <<create>>
create sensor
createcontrol -> sensor: <<create>>
loop add virual sound source
    alt add from library
        note right of user: Notice that there are no shared SoundSources here, each time, only the SoundFragment object is reused.
        user -> createboundary: selectExistedSoundSource
        createboundary -> createcontrol: getLocalSoundFragments
        createcontrol -> library: getLocalSoundFragments
        loop for each
            library -> vsses: getSoundSources
            vsses --> library: soundSources
        end
        library --> createcontrol: filteredSoundFragments
        createcontrol --> createboundary: displayAvailableLocalSoundFragments
        user -> createboundary: selectSoundFragment
        createboundary -> createcontrol: addSoundFragment
    else add from file
        user -> createboundary: selectFromFileSystem
        ...somehow MapVSSCreateControl got the new SoundFragment # specify during implementation...

    end

    createcontrol -> sensor: getCurrentLocation
    sensor --> createcontrol: currentLocation

    participant ":SoundSource" as source
    create source
    createcontrol -> source: <<create>>
    createcontrol -> source: setSoundFragment
    createcontrol -> source: setLocation

    alt user choose to alter position
        user -> createboundary: setCurrentSoundSourceLocation
        createboundary -> source: setLocation
    end
    user -> createboundary: setSoundSourceMode
    createboundary -> createcontrol: setSoundSourceMode
    createcontrol -> source: setSoundSourceMode
    createcontrol -> newspace: addSoundSource(soundSource)
    user -> createboundary: setVSSNameAndDescription
    createboundary -> createcontrol: setVSSNameAndDescription
    createcontrol -> newspace: setVSSNameAndDescription
    user -> createboundary: comfirmVSSCreation
    createboundary -> createcontrol: comfirmVSSCreation
    createcontrol -> library: addSoundSpace(newSoundSpace)
    createcontrol -> createboundary: <<destroy>>
    destroy createboundary
    createcontrol -> sensor: <<destroy>>
    destroy sensor
    destroy createcontrol


end
@enduml
```

## 6.CreateConcertModeVSS

![SeqDiagram](./CreateConcertModeVSS.svg)

```PlantUML
@startuml CreateConcertModeVSS
hide footbox
skinparam sequenceParticipant underline


actor User as user
participant ":ConcertVSSCreatePage" as createboundary
participant ":ConcertVSSCreateControl" as createcontrol
participant ":LocalSoundSpaceLibrary" as library
collections "soundSpaces:VirtualSoundSpace" as vsses
participant "newSoundSpace:ConcertModeVirtualSoundSpace" as newspace

create createboundary
-> createboundary: <<create>>
create createcontrol
createboundary -> createcontrol: <<create>>
create newspace
createcontrol -> newspace: <<create>>

loop add virual sound source
    alt add from library
        note right of user: Notice that there are no shared SoundSources here, each time, only the SoundFragment object is reused.
        user -> createboundary: selectExistedSoundSource
        createboundary -> createcontrol: getLocalSoundFragments
        createcontrol -> library: getLocalSoundFragments
        loop for each
            library -> vsses: getSoundSources
            vsses --> library: soundSources
        end
        library --> createcontrol: filteredSoundFragments
        createcontrol --> createboundary: displayAvailableLocalSoundFragments
        user -> createboundary: selectSoundFragment
        createboundary -> createcontrol: addSoundFragment
    else add from file
        user -> createboundary: selectFromFileSystem
        ...somehow MapVSSCreateControl got the new SoundFragment # specify during implementation...
    end


    participant ":SoundSource" as source
    user -> user: wakeUp(self)
    create source
    createcontrol -> source: <<create>>
    createcontrol -> source: setSoundFragment
    createcontrol -> newspace: addSoundSource(soundSource)
    user -> createboundary: setVSSNameAndDescription
    createboundary -> createcontrol: setVSSNameAndDescription
    createcontrol -> newspace: setVSSNameAndDescription
    user -> createboundary: comfirmVSSCreation
    createboundary -> createcontrol: comfirmVSSCreation
    createcontrol -> library: addSoundSpace(newSoundSpace)

    createcontrol -> createboundary: <<destroy>>
    destroy createboundary
    destroy createcontrol


end
@enduml
```

## 7.RenameVSS

![SeqDiagram](./RenameVSS.svg)

```PlantUML
@startuml RenameVSS
hide footbox
skinparam sequenceParticipant underline

actor User as user
participant ":LocalVSSManagerPage" as page
participant ":LocalVSSLibraryControl" as control
participant "whichVSS:VirtualSoundSpace" as vss

user -> page: renameVSS
user -> page: enterNewLocalVSSName
note right of control: we directly store the VSS's references from library in LocalVSSLibraryControl
page -> control: newNameEntered(whichVSS)
control -> vss: setName(newName)

@enduml
```

## 8.DeleteVSS
![SeqDiagram](./DeleteVSS.svg)

```PlantUML
@startuml DeleteVSS
hide footbox
skinparam sequenceParticipant underline

actor User as user
participant ":LocalVSSManagerPage" as page
participant ":LocalVSSLibraryControl" as control
participant "whichVSS:VirtualSoundSpace" as vss
participant ":LocalSoundSpaceLibrary" as library

user -> page: deleteVSS(whichVSS)
page -> control: deleteVSS(whichVSS)

control -> vss: destroy
vss -> library: unregister(self)
destroy vss

@enduml
```

## 9.UploadVSS
![SeqDiagram](./UploadVSS.svg)

```PlantUML
@startuml UploadVSS
hide footbox
skinparam sequenceParticipant underline

actor User as user
participant ":LocalVSSManagerPage" as page
participant ":LocalVSSLibraryControl" as control
participant "whichVSS:VirtualSoundSpace" as vss
participant ":LocalSoundSpaceLibrary" as library

user -> page: uploadVSS(whichVSS)
page -> control: uploadVSS(whichVSS)

control -> vss: upload
vss -> library: beamMeUp(whichVSS)
...LocalSoundSpaceLibrary doing magic trick, using android system to show download progress...
...There are no other ways to show that a local sound space is finished uploading other than android system notices...
@enduml
```

## 10. LikeVSS
![SeqDiagram](./LikeVSS.svg)

```PlantUML
@startuml LikeVSS
hide footbox
skinparam sequenceParticipant underline
actor User as user
participant ":PreviewOnline***Page" as page
participant ":PreviewOnline***Control" as control
participant ":UserInfoControl" as info

user -> page: likeVSS
page -> control: likeVSS
control -> info: getCurrentUserID()
info --> control: currentUserID
control -> vss: likedBy(currentUserID)
participant ":Like" as like
create like
vss -> like: <<create>>

@enduml
```


## 11. DownloadVSS
![SeqDiagram](./DownloadVSS.svg)

```PlantUML
@startuml DownloadVSS
hide footbox
skinparam sequenceParticipant underline
actor User as user
participant ":PreviewOnline***Page" as page
participant ":PreviewOnline***Control" as control
participant "vss:VirtualSoundSpace" as vss
participant "cloneVSS:VirtualSoundSpace" as clone
participant ":LocalSoundSpaceLibrary" as library
user -> page: downloadVSS
note right of vss: download operation downloads every SoundSource's SoundFragment, a separate local VSS is created.
note right of vss: every SoundFragment is identified uniquely by MD5 of the file.
page -> control: downloadVSS
control -> vss: download
create clone
vss -> clone: <<create>>
vss -> clone: copyMetaDataFrom(vss)
note right
MetaData stands for all forms of data only that a VSS contains,
including SS, SF, but the SFs are not downloaded.
notice that we copy the references for SoundSources, it should be OK.
end note
vss -> library: addSoundSpace(cloneVSS)
vss -> clone: downloadAll
note right
downloadAll makes cloneVSS to attempt to download all its SoundFragments
end note
loop for each soundFragment in cloneVSS
    clone -> library: checkSoundFragmentExist(soundFragment)
    alt exist
        library -> clone: sameDownloadedSoundFragment
    else nonexist
        library -> clone: null
    end
    note right
        because we want to have simpler data structure, we got complex procedures
    end note
end



@enduml
```

## 12. CommentVSS
![SeqDiagram](./CommentVSS.svg)

```PlantUML
@startuml CommentVSS
hide footbox
skinparam sequenceParticipant underline
actor User as user
participant ":PreviewOnline***Page" as page
participant ":PreviewOnline***Control" as control
participant ":UserInfoControl" as info
participant "vss:VirtualSoundSpace" as vss

user -> page: submitComment(comment: String)
page -> control: submitComment(commit: String)
control -> info: getCurrentUserID()
info --> control: currentUserID
control -> vss: addCommit(userid, comment)
participant ":Comment" as comment
create comment
vss -> comment: <<create>>
@enduml
```


## 13. PlayVSS(Abstract)

## 14. PreviewVSS(Abstract)

## 15. PreviewLocalMapVSS
![SeqDiagram](./PreviewLocalMapVSS.svg)

```PlantUML
@startuml PreviewLocalMapVSS
hide footbox
skinparam sequenceParticipant underline
participant ":PreviewLocalMapPage" as page
participant ":PreviewLocalMapControl" as control
participant "localMapVSS:VirtualSoundSpace" as vss
create page
[-> page: <<create>>
create control
page -> control: <<create>>
control -> vss: get metadata
vss --> control: metadata
control -> page: display metadata



@enduml
```


## 16. PreviewOnlineMapVSS
![SeqDiagram](./PreviewOnlineMapVSS.svg)

```PlantUML
@startuml PreviewOnlineMapVSS
hide footbox
skinparam sequenceParticipant underline
actor User as user

participant ":PreviewOnlineMapPage" as page
participant ":PreviewOnlineMapControl" as control
participant "onlineMapVSS:VirtualSoundSpace" as vss
create page
[-> page: <<create>>
create control
page -> control: <<create>>
control -> vss: get metadata
vss -> control: metadata
control -> page: display metadata
note right
details are left out for designs.
end note
@enduml
```

## 17. PreviewLocalConcertVSS
![SeqDiagram](./PreviewLocalConcertVSS.svg)

```PlantUML
@startuml PreviewLocalConcertVSS
hide footbox
skinparam sequenceParticipant underline
actor User as user

participant ":PreviewLocalConcertPage" as page
participant ":PreviewLocalConcertControl" as control
participant "localConcertVSS:VirtualSoundSpace" as vss
create page
[-> page: <<create>>
create control
page -> control: <<create>>
control -> vss: get metadata
vss -> control: metadata
control -> page: display metadata
note right
details are left out for designs.
end note
@enduml
```


## 18. PreviewOnlineConcertVSS
![SeqDiagram](./PreviewOnlineConcertVSS.svg)

```PlantUML
@startuml PreviewOnlineConcertVSS
hide footbox
skinparam sequenceParticipant underline
actor User as user

participant ":PreviewConcer" as page
participant ":PreviewOnlineConcertPage" as control
participant "onlineConcertVSS:VirtualSoundSpace" as vss
create page
[-> page: <<create>>
create control
page -> control: <<create>>
control -> vss: get metadata
vss -> control: metadata
control -> page: display metadata
note right
details are left out for designs.
end note
@enduml
```


## 19. PlayMapModeVSS
![SeqDiagram](./PlayMapModeVSS.svg)

```PlantUML
@startuml PlayMapModeVSS

@enduml
```


## 20. PlayConcertModeVSS
![SeqDiagram](./PlayConcertModeVSS.svg)

```PlantUML
@startuml PlayConcertModeVSS
hide footbox
skinparam sequenceParticipant underline
actor User as user

@enduml
```


## 21. BrowseOnlineVSSLibrary
![SeqDiagram](./BrowseOnlineVSSLibrary.svg)

```PlantUML
@startuml BrowseOnlineVSSLibrary

@enduml
```

## 22. AdjustSensor
![SeqDiagram](./AdjustSensor.svg)

```PlantUML
@startuml AdjustSensor

@enduml
```
