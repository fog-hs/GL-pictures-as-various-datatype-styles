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

here, a "Picture" is a pair of a postion (Pos) and a PictureType representing various types of picture.
the consideration which motivated this pairing with postition was noting that GL pictures can be "translated" but without having first fixed them to a canvas of given dimensions.
then it is natural to ask if there are any other properties that can be placed in this tuple of properties associated to a particular PictureType, where Colour seems like a perfect example, except that it is not natural to associate a colour to a bitmap... how can we resolve this?

```
type Colour = (Int,Int,Int)
```
the first way we could approach this would be just to have;

```
data Picture = Picture {picturePlacement :: Pos,pictureColour :: Colour,getPicture :: PictureType}
```

but this is no good, here we would associate a colour to a bitmap needlessly.
we could try;

```
data Picture = Picture {picturePlacement :: Pos,pictureColour :: Maybe Colour,getPicture :: PictureType}
```

and just have `Nothing :: Maybe Colour` to be the expected thing associated with a BMP. 
but then the user still has the option of providing a Colour, where really we want to somehow indicate that BMP *has* to be paired with Nothing. here we can use a phantom type of kind Bool to indicate if the Picture has a Colour or not, and use a GADT constructor to ensure the type returned by the BMP constructor is False, and have a singletons version of Maybe for storing the colour so it can be specified by this type level Bool that it is Nothing for BMPs.

```
data SMaybe (isJust :: Bool) a where
 Nothing ::      SMaybe 'False a
 Just    :: a -> SMaybe 'True  a

data Picture (hasColour :: Bool) = Picture {picturePlacement :: Pos,
                                            pictureColour :: SMaybe hasColour Colour,
                                            getPicture :: PictureType hasColour}

data PictureType (hasColour :: Bool) where
 Point :: Int        -> PictureType 'True
 Line  :: Pos -> Pos -> PictureType 'True
 BMP   :: (Int,Int) -> ((Int,Int) -> (Int,Int,Int))
                     -> PictureType 'False
```

then, we could have the phantom type take lables corresponding to each of the constructors, and have a type family to specify the association of each picture type to the Boolean indicating if it has a colour - a HasColour type family taking these lables as arguments and returning types of kind Bool. this type family can then be used in place of the Maybe Colour to specify if it is Just or Nothing. 
This is the coding style that this post was supposed to demonstrate. having a datatype of labels corresponding to the constructors which can be used with a type family as an argument to fix the types which depend on which Constructor is being used.  


