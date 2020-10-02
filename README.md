# Metin2 Dynamic Item DropList Window
What is going on?
Whenever I was playing on any server that had far too many drop items from monsters — I wondered — how am I going to find myself in this pile of scrap?
You click on one item and pick up a completely different one, thus littering your inventory.

With my solution, your players don't have to worry about it anymore.

# As a player
Pros:
* You don't have to worry about items gained by other players
* The alphabetical list will allow you to easily find the acquired item
* The dynamic list cleans and completes itself on a regular basis
* You do not have to worry that your list will be flooded with items that have been lying on the ground for a long time — only the items you have acquired will go to the list
* You can double-click on the name of the item and the character will automatically move towards it with the intention of picking up
* You can refresh the list by yourself
* You can select an item from the list and pick it up from the ground with one button
#real-cool-heading
Min:
* You still need to be close to the item to pick it up — but it's probably fair.
# Gif
>(may be displayed once)

[![1](http://nirray.bplaced.net/Download/Github/m2/1.gif)](http://nirray.bplaced.net/Download/Github/m2/1.gif)
[![2](http://nirray.bplaced.net/Download/Github/m2/2.gif)](http://nirray.bplaced.net/Download/Github/m2/2.gif)


>YouTube video explanation :

[![YouTubeVideo](https://img.youtube.com/vi/M5Se5fqgxkE/0.jpg)](https://www.youtube.com/watch?v=M5Se5fqgxkE)

# Installation
* [Python uidroplist.py](#uidroplist)
* [Python constInfo.py](#constInfo)
* [Python game.py](#game)
* [Python droplistwindow](#droplistwindow)
* [locale_game.txt and locale_interface.txt](#locale)
* [C++ locale_inc.h](#locale_inc)
* [C++ PythonPlayer](#pythonplayer)

##### Client part
# uiDropList
1. Place **uidroplist.py** inside *root* directory.
# ConstInfo
2. Open **constInfo.py** inside *root* directory:
#### Search:
```python
PVPMODE_PROTECTED_LEVEL = 15
```
#### Add below:
```python
DROPLIST_VID_AND_ITEM_NAME = {}
OWN_ITEM_VID = []
DROP_LIST_REFRESH = 0
DROP_LIST_LAST = ""
DROP_LIST_DOUBLE = 0
DROP_LIST_SLIDER = 0
```
#### Search def:
```python
def SET_TWO_HANDED_WEAPON_ATT_SPEED_DECREASE_VALUE():
	global TWO_HANDED_WEAPON_ATT_SPEED_DECREASE_VALUE
	app.SetTwoHandedWeaponAttSpeedDecreaseValue(TWO_HANDED_WEAPON_ATT_SPEED_DECREASE_VALUE)
```
#### Add below:
```python
def DropListAppend(vid, item_name):
	global DROPLIST_VID_AND_ITEM_NAME
	append = {vid : item_name}
	DROPLIST_VID_AND_ITEM_NAME.update(append)

def DropListMyItem(vid):
	global OWN_ITEM_VID
	global DROP_LIST_REFRESH
	OWN_ITEM_VID.append(vid)
	DROP_LIST_REFRESH = 1

def RemoveFromDropList(vid):
	global DROPLIST_VID_AND_ITEM_NAME
	global OWN_ITEM_VID
	global DROP_LIST_REFRESH
	if DROPLIST_VID_AND_ITEM_NAME and OWN_ITEM_VID:
		if vid in DROPLIST_VID_AND_ITEM_NAME:
			del DROPLIST_VID_AND_ITEM_NAME[vid]
			OWN_ITEM_VID.remove(vid)
			DROP_LIST_REFRESH = 1
```
# Game
3. Open **game.py** inside *root* directory:

#### Search:
```python
import interfaceModule
```

#### Add below:
```python
import uidroplist
```
#### Search:
```python
	self.quickSlotPageIndex = 0
```
#### Add below:
```python
	self.dropListDlg = 0
```

#### Search:
```python
	def __BuildKeyDict(self):
```
#### Below:
```python
	onPressKeyDict[app.DIK_F4]	= lambda : self.__PressQuickSlot(7)
```
#### Add:
```python
	onPressKeyDict[app.DIK_F5]	= lambda : self.ScanDropList()
```
>You can change Key DIK if you want to

#### Search:
```python
	def StopRight(self):
		player.SetSingleDIKKeyState(app.DIK_RIGHT, False)
```

#### Add below:
```python
	def ScanDropList(self):
		if not self.dropListDlg:
			self.dropListDlg=uidroplist.FileListDialog()
		self.dropListDlg.Open()

	def DropListUpdate(self, vid, name):
		constInfo.DropListAppend(int(vid), str(name))

	def DropListOwn(self, vid):
		constInfo.DropListMyItem(int(vid))

	def RemoveVidFromList(self, vid):
		constInfo.RemoveFromDropList(int(vid))
```

# DropListWindow
4. Place **droplistwindow.py** inside *uiscript* directory.

# Locale
5. Paste the relevant entries in **locale_interface.txt** and **locale_game.txt** inside *locale* directory.

##### Binary part
# Locale_inc
6. Open **locale_inc.h** inside *UserInterface* directory:

#### Add:
```cpp
#define ENABLE_DROPLIST_WINDOW
```

# PythonPlayer
7. Open **PythonPlayer.h** inside *UserInterface* directory:
#### Search:
```cpp
	void	SendClickItemPacket(DWORD dwIID);
```

#### Add below:
```cpp
#ifdef ENABLE_DROPLIST_WINDOW
	void	SendClickItemPacketDropList(DWORD dwIID);
#endif
```

#### Search:
```cpp
		void	ClearSkillDict();
```

#### Add below:
```cpp
#ifdef ENABLE_DROPLIST_WINDOW
		void	DropListAppend(DWORD VID, string Item_Name);
		void	DropListOwn(DWORD VID);
		void	RemoveFromOwnList(DWORD VID);
#endif
```

8. Open **PythonPlayer.cpp** inside *UserInterface* directory:

#### Search:
```cpp
void CPythonPlayer::ClearSkillDict()
{
	...
}
```

#### Add below:
```cpp
#ifdef ENABLE_DROPLIST_WINDOW
void CPythonPlayer::DropListAppend(DWORD VID, string Item_Name)
{
	PyCallClassMemberFunc(m_ppyGameWindow, "DropListUpdate", Py_BuildValue("(is)", VID, Item_Name.c_str()));
}
void CPythonPlayer::DropListOwn(DWORD VID)
{
	PyCallClassMemberFunc(m_ppyGameWindow, "DropListOwn", Py_BuildValue("(i)", VID));
}
void CPythonPlayer::RemoveFromOwnList(DWORD VID)
{
	PyCallClassMemberFunc(m_ppyGameWindow, "RemoveVidFromList", Py_BuildValue("(i)", VID));
}
#endif
```

#### Search:
```cpp
DWORD CPythonPlayer::GetPlayTime()
{
	return m_dwPlayTime;
}
```

#### Add below:
```cpp
#ifdef ENABLE_DROPLIST_WINDOW
void CPythonPlayer::SendClickItemPacketDropList(DWORD dwIID)
{
	CInstanceBase* pkInstMain = NEW_GetMainActorPtr();
	if (!pkInstMain) 
		return;
	if (IsObserverMode())
		return;
	if (pkInstMain->IsStun())
		return;
	if (pkInstMain->IsDead())
		return;
	CPythonItem& rkIT = CPythonItem::Instance();

	TPixelPosition kPPosPickedItem;
	if (rkIT.GetGroundItemPosition(dwIID, &kPPosPickedItem))
	{
		float distance = 120.0f;
		if (pkInstMain->IsMountingHorse() || pkInstMain->IsNewMount())
			distance = 150.0f;
		if (pkInstMain->NEW_GetDistanceFromDestPixelPosition(kPPosPickedItem) < distance)
		{
			CPythonNetworkStream& rkNetStream = CPythonNetworkStream::Instance();
			TPixelPosition kPPosCur;
			pkInstMain->NEW_GetPixelPosition(&kPPosCur);
			float fCurRot = pkInstMain->GetRotation();
			rkNetStream.SendCharacterStatePacket(kPPosCur, fCurRot, CInstanceBase::FUNC_WAIT, 0);
			pkInstMain->NEW_Stop();
			__ClearReservedAction();
			SendClickItemPacket(dwIID);
		}
		else
		{
			pkInstMain->NEW_MoveToDestPixelPositionDirection(kPPosPickedItem);
			SendClickItemPacket(dwIID);
		}
	}
	else
	{
		__ClearReservedAction();
		SendClickItemPacket(dwIID);
	}
}
#endif
```

# PythonTextTail
Important :exclamation:
If you're using xP3NG3Rx **[C++] ItemName reneval on the ground** from [here](https://metin2.dev/board/topic/17399-c-itemname-reneval-on-the-ground/)
