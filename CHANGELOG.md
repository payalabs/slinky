# Changelog

## vNEXT
+ **BREAKING**: Instead of taking key and refs as additional parameters next to props, they are now taken in through the methods `withKey` and `withRef` (components and external components only)
+ **BREAKING**: Introduce the macro annotation `@react` to simplify component and external component creation with auto-generated companion object for a component class (or external component object). This is a major change to how applications with Slinky are written, so please see the notes at the end of the changelog [PR #29](https://github.com/shadaj/slinky/pull/29)
+ **BREAKING**: Rename `ExternalComponentWithTagMods` to `ExternalComponentWithAttributes` and take attributes as a curried parameter instead of an extra parameter after `Props` [PR #26](https://github.com/shadaj/slinky/pull/26)
+ Have mouse attributes such as `onMouseDown` take a `MouseEvent` instead of just an `Event` [PR #27](https://github.com/shadaj/slinky/pull/27)

### `@react` macro annotation
One of Slinky's main goals is to have React components written in Scala look very similar to ES6. In version 0.1.x, Slinky required
extra boilerplate for defining an object that contained `apply` methods and then creating a `Definition` inner class that contained the actual component logic.

This version includes the `@react` macro annotation, which makes it possible to only write the class containing component logic and have Slinky generate
the companion object for constructing component instances. The macro annotation also now generates special `apply` methods when your Props is a case class
so that constructing Scala components looks more similar to JSX, with the Props values directly taken as parameters of the `apply`.

Note that use of the macro annotation is **not required**. To use the original component style simply replace the `extends Component` to `extends ComponentWrapper` and your
components should continue to function as they did before.

As an example of migrating an existing component to the new macro annotation style, take a simple component that displays a header:
```scala
import me.shadaj.slinky.core.WrapperComponent
import me.shadaj.slinky.web.html._

object HelloMessage extends WrapperComponent {
  case class Props(name: String)
  type State = Unit

  @ScalaJSDefined
  class Def(jsProps: js.Object) extends Definition(jsProps) {
    def render() = {
      div(s"Hello ${props.name}")
    }
  }
}
``` 

to use the new macro annotation style, we essentially extract out the definition class, move the `Props` and `State` types into the class, and extend `Component` instead of `Definition`:
```scala
import me.shadaj.core.{Component, react}
import me.shadaj.slinky.web.html._

@react class HelloMessage extends Component {
  case class Props(name: String)
  type State = Unit
  
  def render() = {
    div(s"Hello ${props.name}")
  }
}
```

If we want to use this component, we now have a new option for constructing it directly passing in the Props values
```scala
HelloMessage(HelloMessage.Props("Shadaj")) // old style
HelloMessage("Shadaj") // now possible!
HelloMessage(name = "Shadaj") // now possible, closest to JSX
```

The `@react` annotation is also available for external components. For external components, the annotation generates the new `apply` method style in the same style as Scala components.
```scala
import me.shadaj.slinky.core.annotations.react
import me.shadaj.slinky.core.ExternalComponent

@react object React3 extends ExternalComponent {
  case class Props(mainCamera: String, width: Int, height: Int,
                   onAnimate: Option[() => Unit] = None, alpha: Boolean = false)

  override val component: js.Object = js.Dynamic.global.React3.asInstanceOf[js.Object]
}
```

this makes it possible to construct the external component as
```scala
React3(mainCamera = "camera", width = 800, height = 800)
```

## v0.1.1
+ Have ExternalComponentsWithTagMods take the tag type as a type parameter instead of an abstract type [PR #19](https://github.com/shadaj/slinky/pull/19)
+ Added support for reading and writing values of type `js.UndefOr[T]` [PR #18](https://github.com/shadaj/slinky/pull/18)
+ Components and external components with a `Props` type of `Unit` can now be constructed without any parameters, instead of having to pass in `()` as props [PR #12](https://github.com/shadaj/slinky/pull/12)
+ Boolean attributes, such as `disabled`, can now be used without specifying a value to closer match JSX. For example, a disabled input can now be constructed as `input(disabled)` without providing the `:= true` [PR #14](https://github.com/shadaj/slinky/pull/14)

## v0.1.0
+ Initial release
