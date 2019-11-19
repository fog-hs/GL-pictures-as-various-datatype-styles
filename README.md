# GL-pictures-as-various-datatype-styles

OpenGL has a "picture" object.
It is a primitive which exists without a "canvas" with pixle dimensions,
which means it should have a position field along with each different type of picture,
so that at some point it can be rastered to a bitmap.

```
> type Pos = (Int,Int)

> data Picture = Picture {picturePlacement :: Pos,getPicture :: PictureType}

> data PictureType = Point Int
>                  | Line Pos Pos
>                  | BMP  (Int,Int) ((Int,Int) -> (Int,Int,Int))
```
