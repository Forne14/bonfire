EN | [PT](https://github.com/RafaelBarbosatec/bonfire/blob/master/README_PT.md)

[![Open Source Love](https://badges.frapsoft.com/os/v1/open-source.svg?v=102)](https://github.com/RafaelBarbosatec/bonfire)
[![Powered by Flame](https://img.shields.io/badge/Powered%20by-%F0%9F%94%A5-orange.svg)](https://flame-engine.org)
[![Flutter](https://img.shields.io/badge/Made%20with-Flutter-blue.svg)](https://flutter.dev/)
[![MIT Licence](https://badges.frapsoft.com/os/mit/mit.svg?v=103)](https://opensource.org/licenses/mit-license.php)
[![pub package](https://img.shields.io/pub/v/bonfire.svg)](https://pub.dev/packages/bonfire)

![](https://github.com/RafaelBarbosatec/bonfire/blob/master/media/bonfire.gif)

# Bonfire

Build RPG games and similar with the power of [FlameEngine](https://flame-engine.org/)!

![](https://github.com/RafaelBarbosatec/bonfire/blob/master/media/video_example.gif)

[Download Demo](https://github.com/RafaelBarbosatec/bonfire/raw/master/demo/demo.apk)

Find the complete code of this example [here](https://github.com/RafaelBarbosatec/bonfire/tree/master/example).

## Summary
1. [How it works?](#how-it-works)
   - [Map](#map)
   - [Decorations](#decorations);
   - [Enemy](#enemy)
   - [Player](#player)
   - [Interface](#interface)
   - [Joystick](#joystick)
4. [Utility Components](#utility-components)
3. [Next steps](#next-steps)

## How it works?

This tool was built over [FlameEngine](https://flame-engine.org/) and all its resources and classes are available to be used along with Bonfire. With that said, it is recommended to give a look into [FlameEngine](https://flame-engine.org/) before start rocking with Bonfire.  

To run a game with Bonfire, use the following widget:

```dart
@override
  Widget build(BuildContext context) {
    return BonfireWidget(
      joystick: MyJoystick(), // required
      map: DungeonMap.map(), // required
      player: Knight(), // If player is omitted, the joystick directional will control the map view, being very useful in the process of building maps
      interface: KnightInterface(),
      decorations: DungeonMap.decorations(),
      enemies: DungeonMap.enemies(),
      background: BackgroundColorGame(Colors.blueGrey[900]),
      constructionMode: false, // If true, activates hot reload to ease the map constructions and draws the grid
      showCollisionArea: false, // If true, show collision area of the elements
      gameController: GameController() // If you want to hear changes to the game to do something.
    );
  }
```

Components description and organization:

![](https://github.com/RafaelBarbosatec/bonfire/blob/master/media/game_diagram.png)

### Map
Represents a map (or world) where the game occurs

It is a  matrix of small tiles that toghether assembles the map [(see)](https://www.mapeditor.org/img/screenshot-terrain.png). Right now the matrix is created manually, but in the future it will be possible to load maps created with [Tiled](https://www.mapeditor.org/)

There is a component for this: 
```dart
MapWorld(List<Tile>())
```

MapWorld receives a list of tiles that will assemble our map. The whole camera movimentation during Player actons are included on it. 

```dart
Tile(
   'tile/wall_left.png', // Tile image
   Position(positionX, positionY), // Map coordinates of this tile
   collision: true, // Define if this tile will be not transpassable by players and enemies (ideal for walls and obstacles)
   size: 16 // Tile size (width and height)
)
```

### Decorations
Anything that you may add to the scenery. For example a Barrel in the way or even a NPC in which you can use to interact with your player.

To create a decoration:

```dart
GameDecoration(
  spriteImg: 'itens/table.png', // Image to be rendered
  initPosition: getRelativeTilePosition(10, 6), // World coordinates in which this decoration will be positioned
  width: 32,
  height: 32,
  withCollision: true, // Adds a default collision area
  collision: Collision( // A custom collision area
    width: 18,
    height: 32,
  ),
//  isTouchable: false, // if you want this component to receive touch interaction. You will be notified at 'void onTap()'
//  animation: FlameAnimation(), // Optional param to create an animated decoration. When using this, do not specify spriteImg.
//  frontFromPlayer: false // Define true if this decoration shall be rendered above the Player
)
```   

You can also create your own decoration class by extending `GameDecoration` and implement `update` and `render`  methods with your own behavior. As this [example](https://github.com/RafaelBarbosatec/bonfire/blob/master/example/lib/decoration/chest.dart): A treasure chest that opens when a player gets close, removes itself from the game and puts two life potions in its place (being the life portions a `GameDecoration` as well).

In this component (like all others), you have access to `BuildContext` of the game widget. Therefore, is possible to opebn dialogis, show overlays and other Flutter components that may depend on that.  

### Enemy
Represents enemies characters in the game. Instances of this class has actions and movements ready to be used and configured whenever you want. At the same time, you can customize  all actions and movements in the way that fits your needs.

To create an enemy you shall create an `Enemy` subclass to represent it. Like in this [example](https://github.com/RafaelBarbosatec/bonfire/blob/master/example/lib/enemy/goblin.dart).

The constructor looks like:
```dart
Goblin() : super(
          animationIdleRight: FlameAnimation(), //required
          animationIdleLeft: FlameAnimation(), // required
          animationIdleTop: FlameAnimation(),
          animationIdleBottom: FlameAnimation(),
          animationRunRight: FlameAnimation(), //required
          animationRunLeft: FlameAnimation(), //required
          animationRunTop: FlameAnimation(),
          animationRunBottom: FlameAnimation(),
          initDirection: Direction.right,
          initPosition: Position(x,y),
          width: 25,
          height: 25,
          speed: 1.5,
          life: 100,
          collision: Collision(), // A custom collision area
        );
```   

After these steps, the enemy is ready, but it will stay still. To add movements and behaviors, you shall implement them on the `update` method.

There is already some pre included actions that you can use (as seen on this [example](https://github.com/RafaelBarbosatec/bonfire/blob/master/example/lib/enemy/goblin.dart)), they are:


```dart

//basic movements
void moveBottom({double moveSpeed})
void moveTop({double moveSpeed})
void moveLeft({double moveSpeed})
void moveRight({double moveSpeed})
    
  // Will observe the player when within the radius (visionCells)
  void seePlayer(
        {
         Function(Player) observed,
         Function() notObserved,
         int visionCells = 3,
        }
  )
  
  // Will move in the direction of the player once it gets close within the visibleCells radius . Once it gets to the player, `closePlayer` shall be fired 
  void seeAndMoveToPlayer(
     {
      Function(Player) closePlayer,
      int visionCells = 3
     }
  )
  
  // Executes a physical attack to the player, making the configured damage with the configured frequency. You can add animations to represent this attack.
  void simpleAttackMelee(
     {
       @required double damage,
       @required double heightArea,
       @required double widthArea,
       int interval = 1000,
       FlameAnimation.Animation attackEffectRightAnim,
       FlameAnimation.Animation attackEffectBottomAnim,
       FlameAnimation.Animation attackEffectLeftAnim,
       FlameAnimation.Animation attackEffectTopAnim,
     }
  )

  // Executes a distance attack. Will add a `FlyingAttackObject` to the game and will be send in the configures direction and will make some damage to whomever it hits, or be destroyed as it hits barriers (collision defined tiles).
  void simpleAttackRange(
     {
       @required FlameAnimation.Animation animationRight,
       @required FlameAnimation.Animation animationLeft,
       @required FlameAnimation.Animation animationTop,
       @required FlameAnimation.Animation animationBottom,
       @required FlameAnimation.Animation animationDestroy,
       @required double width,
       @required double height,
       double speed = 1.5,
       double damage = 1,
       Direction direction,
       int interval = 1000,
     }
  )
  // Will seek for the player in the defined radius. When the player is found, will position itself to perform a distance attack. Once it reaches the attack position, will fire the `positioned` callback.
  void seeAndMoveToAttackRange(
      {
        Function(Player) positioned,
        int visionCells = 5
      }
  )
  
  // Exibe valor do dano no game com uma animação.
   void showDamage(
      double damage,
      {
         TextConfig config = const TextConfig(
           fontSize: 10,
           color: Colors.white,
         )
      }
    )
    
    // Add to `render` method if you want to draw the collision area.
    void drawPositionCollision(Canvas canvas)
    
    // Gives the direction of the player in relation to this enemy
    Direction directionThatPlayerIs()
    
    // Executes an animation once.
    void addFastAnimation(FlameAnimation.Animation animation)
    
    // Applies damage to the enemy
    void receiveDamage(double damage)
    
    // Restore life point to the enemy
    void addLife(double life)
  
    // Add to 'render' if you want to draw the collision area
    void drawPositionCollision(Canvas canvas)


    // Draws the default life bar, Should be used in the `render` method.
    void drawDefaultLifeBar(
      Canvas canvas,
      {
        bool drawInBottom = false,
        double padding = 5,
        double strokeWidth = 2,
      }
    )
    
```

### Player
Represents the character controlled by the user in the game. Instances of this class has actions and movements ready to be used and configured.

To create an enemy you shall create an `Player` subclass to represent it. Like in this [example](https://github.com/RafaelBarbosatec/bonfire/blob/master/example/lib/player/knight.dart).

The constructor looks like:
```dart
Knight() : super(
          animIdleLeft: FlameAnimation(), // required
          animIdleRight: FlameAnimation(), //required
          animIdleTop: FlameAnimation(),
          animIdleBottom: FlameAnimation(),
          animRunRight: FlameAnimation(), //required
          animRunLeft: FlameAnimation(), //required
          animRunTop: FlameAnimation(),
          animRunBottom: FlameAnimation(),
          width: 32,
          height: 32,
          initPosition: Position(x,y), //required
          initDirection: Direction.right,
          life: 200,
          speed: 2.5,
          collision: Collision(), // A custom collision area
        );
```   

Player instances can receive action configured on the Joystick (read more about it below) by overriding the following method:

```dart
  @override
  void joystickAction(int action) {}
```

Actions can be fired when a jopystck action is received. Just like `Enemy`, here we have some pre-included actions:

```dart
  
  // Executes a physical attack to the player, making the configured damage with the configured frequency. You can add animations to represent this attack.
  void simpleAttackMelee(
     {
       @required FlameAnimation.Animation attackEffectRightAnim,
       @required FlameAnimation.Animation attackEffectBottomAnim,
       @required FlameAnimation.Animation attackEffectLeftAnim,
       @required FlameAnimation.Animation attackEffectTopAnim,
       @required double damage,
       double heightArea = 32,
       double widthArea = 32,
     }
  )
  
  // Executes a distance attack. Will add a `FlyingAttackObject` to the game and will be send in the configures direction and will make some damage to whomever it hits, or be destroyed as it hits barriers (collision defined tiles).
  void simpleAttackRange(
     {
       @required FlameAnimation.Animation animationRight,
       @required FlameAnimation.Animation animationLeft,
       @required FlameAnimation.Animation animationTop,
       @required FlameAnimation.Animation animationBottom,
       @required FlameAnimation.Animation animationDestroy,
       @required double width,
       @required double height,
       double speed = 1.5,
       double damage = 1,
     }
  )

  // Shows the damage value as an animation on the game.
   void showDamage(
      double damage,
      {
         TextConfig config = const TextConfig(
           fontSize: 10,
           color: Colors.white,
         )
      }
    )
    
    // Will observe enemies when within the radius (visionCells)
    void seeEnemy(
       {
          Function(List<Enemy>) observed,
          Function() notObserved,
          int visionCells = 3,
       }
    )
    
    // Add to `render` method if you want to draw the collision area.
    void drawPositionCollision(Canvas canvas)
    
    // Executes an animation once.
    void addFastAnimation(FlameAnimation.Animation animation)
    
    // Applies damage to the enemy
    void receiveDamage(double damage)
    
    // Restore life point to the enemy
    void addLife(double life)
  
```

### Interface

The way you cand raw things like life bars, stamina and settings. In another words, anything that you may add to the interface to the game.

Interfaces implementations shall be implemented on `GameInterface` subclasses, like this [exemplo](https://github.com/RafaelBarbosatec/bonfire/blob/master/example/lib/player/knight_interface.dart).

Interfaces are drawn by overriding `update` and `render` methods. You can draw directly on canvas or use [FlameEngine](https://flame-engine.org/) components.

### Joystick
The player-controlling component. 

There is a pre-included implementation (`Joystick`) ready to use, but also configurable to add a custom looking or even add as many actions as you will.
Or you can implement `JoystickController` yourself and emit event trough a `JoystickListener`.

Joystick is configurable by the following parameters:
```dart

      Joystick(
        pathSpriteBackgroundDirectional: 'joystick_background.png', //(required) directinal control background
        pathSpriteKnobDirectional: 'joystick_knob.png', //(required) directional indicator circle background
        sizeDirectional: 100, // directional control size
        marginBottomDirectional: 100,
        marginLeftDirectional: 100,
        actions: [     // List of actions that will be placed the screen.
          JoystickAction(
            actionId: 0,      //(required) Action identifier, will be sent to 'void joystickAction(int action) {}' when pressed
            pathSprite: 'joystick_atack.png',     //(required) the action image
            pathSpritePressed : 'joystick_atack.png', // Optional image to be shown when the action is fired
            size: 80,
            margin: EdgeInsets.only(bottom: 50, right: 50),
            align = JoystickActionAlign.BOTTOM_RIGHT,
          ),
          JoystickAction(
            actionId: 1,
            pathSprite: 'joystick_atack_range.png',
            size: 50,
            margin: EdgeInsets.only(bottom: 50, right: 160),
            align = JoystickActionAlign.BOTTOM_RIGHT,
          )
        ],
      )
      
```
n
Check a [example](https://github.com/RafaelBarbosatec/bonfire/blob/master/example/lib/main.dart).

### Observations:

Since all of these elements uses the ´HasGameRef´ mixin, it is possible to acess all components internally. This will be useful for any kind of interaction between elements or the creation of a new one programatically. 

If it is necessary to get the position of a component in the map, use `positionInWorld`.  This is useful for doing some stuff like adding components on the map. While this is the position of the component in relation to the map, `position` is the coordinates relative to the screen. 

## Utility components

Some components with a unique purpose that can be useful. Since any other component that extends Flame's `Component` or Bonfire's `AnimatedObject`, you use it on your game in the following way:

```dart
this.gameRef.add(YOUR_FANCY_COMPONENT);
```

The components are:

```dart

// To run an animation once before it destroys itself
AnimatedObjectOnce(
   {
      Rect position,
      FlameAnimation.Animation animation,
      VoidCallback onFinish,
      bool onlyUpdate = false,
   }
)

// Like the previous one, this can play an animation once before it destroys itself and can also can can keep playing in a loop. But the most important feature is that this component follows another element on the map, like a player, enemy or decoration.
AnimatedFollowerObject(
    {
      FlameAnimation.Animation animation,
      AnimatedObject target,
      Position positionFromTarget,
      double height = 16,
      double width = 16,
      bool loopAnimation = false
   }
)

// Componente que anda em determinada direção configurada em uma determinada velocidade também configurável e somente para ao atingir um inimigo ou player infligindo dano, ou pode se destruir ao atigir algum componente que tenha colisão (Tiles,Decorations).
FlyingAttackObject(
   {
      @required this.initPosition,
      @required FlameAnimation.Animation flyAnimation,
      @required Direction direction,
      @required double width,
      @required double height,
      FlameAnimation.Animation this.destroyAnimation,
      double speed = 1.5,
      double damage = 1,
      bool damageInPlayer = true,
      bool damageInEnemy = true,
  }
)
  
```

If it is necesssary to add a instance of a Bonfire's basic component class (Decorations or Enemy), one shall use its specific methods:
```dart
this.gameRef.addEnemy(ENEMY);
this.gameRef.addDecoration(DECORATION);
```

### Camera

It is possible to move the camera to some position and go back to the player afterwards. Beware that the player will be blocked from taking any action and from moving until the camera has focused on it again.

```dart
 gameRef.gameCamera.moveToPosition(Position(X,Y));
 gameRef.gameCamera.moveToPlayer();
 gameRef.gameCamera.moveToPositionAnimated(Position(X,Y));
 gameRef.gameCamera.moveToPlayerAnimated();
```

## Next steps
- [ ] Component docs
- [ ] [Tiled](https://www.mapeditor.org/) support
- [ ] Using Box2D


## Example game
[![](https://github.com/RafaelBarbosatec/darkness_dungeon/blob/master/icone/icone_small.png)](https://github.com/RafaelBarbosatec/darkness_dungeon)

## Credits

 * The entire FlameEgine team, especially [Erick](https://github.com/erickzanardo).
 * [Renan](https://github.com/renancaraujo) That helped in the translation of the readme.
 * And all those who were able to contribute as they could.
 
