---
title: "Html Dnd Api"
date: 2022-12-11T15:54:13+09:00
draft: true
---

> 이 글은 2022년 12월 11일에 MDN의 [DnD API문서](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API)를 기반하여 작성 되었습니다.

# Drag and Drop API
Drag and Drop API는 DnD 기능을 위해 HTML에서 표준으로 지원하는 browser API입니다. 굉장히 쉬운 API로 별게 없습니다. 이번에 이것을 이용해 간단한 Todo list를 만들고, 이를 제가 일하고 있는 곳에 사용하고자 정리해 봅니다.

## interfaces
HTML의 drag-and-drop 기능은 아래 4가지 object의 interface를 가지고 있습니다. 아래에는 제가 몰랐던 특징들을 위주로 정리해 보았습니다.
1. DragEvent
1. DataTransfer
1. DataTransferItem
1. DataTransferItemList

### DragEvent
1. mouse event를 상속받았습니다.
1. DataTransfer Object를 속성으로 가집니다.

### DataTransfer
1. 옮기고 싶은 정보를 감싸고 있는 object입니다.
1. dropEffect를 설정해 filtering이 가능합니다.
1. items라는 이름으로 DataTransferItemList를 가집니다.
1. items의 모든 MIME type을 속성으로 들고 있습니다.

### DataTransferItem(List)
1. kind라는 속성으로 전달 시점 정보의 형태를 설정하고, type이라는 속성으로 정보의 MIME type을 밝힙니다.
1. custom 동작에는 `text/plain`이 좋다고 합니다.

포함관계는 1 < 2 < 3,4가 됩니다.

## events
drag-and-drop과 관련된 event는 drag 부터 drop까지 총 7개가 있습니다. 

### drag 영역
1. drag
1. dragstart
    - 시작할때 납니다.
    - file을 browser로 옮기는 경우 기본으로는 dragstart/end가 발생하지 않습니다.
    - image를 넣을 수 있습니다. 
    - touch event등을 방지하기 위해 preventDefault 등을 사용할 수 있습니다.
1. dragend
    - cancel되도 납니다.
    - dropEffect가지고 cancel 여부를 판별합니다. (none => cancel)
### drop 영역
1. dragenter
    - 들어오면 항상 발생합니다.
1. dragleave
    - 나가면 항상 발생합니다.
1. dragover
    - 안에 있으면 계속 발생합니다.
1. drop
    - drop 시에만 발생합니다.




