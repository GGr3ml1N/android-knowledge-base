Модификатор shadow позволяет создать тень вокруг элемента. Этот модификатор имеет следующую форму:

fun Modifier.shadow(
    elevation: Dp,
    shape: Shape = RectangleShape,
    clip: Boolean = elevation > 0.dp,
    ambientColor: Color = DefaultShadowColor,
    spotColor: Color = DefaultShadowColor
): Modifier

Функция модификатора принимает следующие параметры:

`elevation`: высота тени в dp
`shape`: определяет форму тени с помощью объекта Shape
clip: если равно true, то создается фрагмент с использованием формы из параметра shape
ambientColor: цвет затенения на компоненте
spotColor: цвет тени вокруг компонента

Для создания тени нам достаточно задать ее высоту.