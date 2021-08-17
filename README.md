ooga
====

This project implements a player for multiple related games.

Names: Donghan Park (dp239), Felix Jiang (fj32), Samy Boutouis (sb590), Kevin Yang (kyy2), Bill Guo (wg78)


### Timeline

Start Date: 3/25/2021

Finish Date: 4/26/2021

Hours Spent: ~400 combined total from all members

### Primary Roles
* Frontend:
    * Samy Boutouis: SeXML parser for dynamic data-driven creation of UI components, player profiles, color themes, multiple views
    * Donghan Park: Game area editor, HUD, timeline manager, game select screens, assets/UI design
* Backend:
    * Felix Jiang: Web, collisions, data, dynamic image changes, other basic features, engine
    * Kevin Yang: Data, data structuring, data parsing, triggers, actions, data and backend testing
    * Bill Guo: Actions, triggers, properties, backend to frontend communication, level and game design

### Resources Used

For sprite animations: [Sprite Animations](https://netopyr.com/2012/03/09/creating-a-sprite-animation-with-javafx/)

### Running the Program

Main class: `src/ooga/Main.java`

Data files needed:
* `src/resources/levels/`
    * In this directory, a directory, LEVELNAME, must be included and hold level data.
    * The level directory (named with the level name) must have:
        * `LEVELNAME.json`, which holds the names of the the depedent data files
        * `LEVELNAMEgrid.json`, which holds the level layout
        * `LEVELNAMEactormap.json`, which holds the map of characters to actortypes in the grid file
        * `LEVELNAMEcollisions.json`, which holds what will happen on collisions
        * `ACTORNAME.json`, several files with some name ACTORNAME which hold what gameobjects do
* `TriggerMap.json`, an internal map that defines methods corresponding to initializing certain triggers
* New CSS stylesheets can be added to `src/resources/stylesheets`; a new key should be written to `src/resources/reflection/UI_Text.properties`
* `src/resources/userInterface/`: user interface files that hold screen and JavaFX components in SeXML file format

Features implemented:

* Frontend:
    * Multiple views: users are able to play multiple games of any game variant at once
    * Player profiles: users are able to create their own profiles; in the profile, users are able to change and save preferences (username, password, profile image, color themes)
    * Game area editor: users can design new levels' initial configuration files dynamically using a GUI in the level editor screen; users can also load existing level files to edit/update the files for existing levels
    * Data-driven using SeXML parsing (all JavaFX components are generated from XML files and are dynamically created using reflection)
        * Any of the UI files can be changed by editing the XML files without changing the Java code
* Backend:
    * Data driven
    * Web saving/loading integration (save files on a web database)
        * Can play any level created from any computer at another computer
        * Data can be converted to files from a web JSON and from files to a web JSON. Allows for any game to be loaded since the parsers agree on how data is formatted.
    * Very flexible and general collisions implementation
    * Provide frontend with changes to image when collisions occur.
    * Parsing data file to create gameobjects and interactions (level, triggers, collisions, etc.)
        * Reads from an already readable format so data files are easy and intuitive to create
    * Dynamic parsing of Data
        * Allows for extreme extension, creating new types of `Trigger` and defining them in `TriggerMap.json` to be created from the `TriggerParser`
    * Save games by writing to files in the required JSON format
        * Creates all necessary new files in a selected directory, with clean packaging to be saved and sent anywhere
    * Dynamic game rules defined by adding defining more actions in the data files
        * All data in actions and triggers can be freely defined in data files, so any amount of different interactions or actions and their required properties.

### Notes/Assumptions

Assumptions or Simplifications:
* There will only be 1 controllable actor in each level at any time.
* Velocities are low enough such that if a collision occurs by checking velocities, we do not move the actors at all. If velocities are high, the character can not be flush against a wall, but we assume our velocities will be low (or frame rate will be high) such that
* Data files follow the data parser format

Interesting data files:
* `src/resources/levels/level3`: this level shows the flexibility in the design by displaying how the direction of movement can be changed
* `src/resources/levels/fishy`: fishy

Known Bugs:
* Based on certain machines' method of displaying text on the screen, the title and texts can be rendered unreadable due to incorrect conversions of text.
* Corner collisions can sometimes render the player stuck, sometimes they can be jumped out of, other times they are stuck for good

Extra credit:
* A whole dynamic system of writing levels to a file was created.
* This would ideally integrate with the web data structure.
* The current system of writing levels to files requires the placeable actors to be stored locally in a single directory.
    * This requires these assets to be preloaded in order to be able to be added to the game.
* With this dynamic system, if a player creates new gameobjects and levels on their local end and another player plays the game online, there is no need for you to install new files and dependencies in order to save a level.
* The writer would take the actor, level, grid, image, etc. information from the game and write that to completely new files for you.
* This takes in the level being currently played (which would be made easier if levels were played directly in the web).
    * This way, the player does not need to manually move data files to the local directory in order to be able to save new types of gameobjects to the game.
* Very flexible collision data parsing, being able to change an entire interaction with single changes in a file.
* Methods to convert JSON to files and files to a JSON, being comfortable with file system manipulation

* Created custom spritesheets & assets
* Custom UI parsing (SXML) that acts similar to HTML
    * Can create any type of JavaFX component with some attributes such as x or y position, width, id, etc.
    * Completely dependent on data files and adding new types of components like buttons or ImageViews are defined in property files


As an example for the code written for the dynamic level file writer:
```java
package ooga.data.writers;

import java.lang.reflect.InvocationTargetException;
import ooga.data.DataParser;
import ooga.engine.LevelIO;
import ooga.engine.actors.Actor;
import org.json.JSONArray;
import org.json.JSONObject;

public class JSONDataFactory extends ReflectiveParser implements DataParser {

  private ActorWriter actorWriter;
  private TriggerWriter triggerWriter;
  private GridWriter gridWriter;
  private CollisionWriter collisionWriter;
  private JSONObject outputFile;

  public JSONDataFactory() {
    actorWriter = new ActorWriter();
    triggerWriter = new TriggerWriter();
    gridWriter = new GridWriter();
    outputFile = new JSONObject();
  }

  public JSONObject save(LevelIO level)
      throws NoSuchMethodException, IllegalAccessException, InvocationTargetException {
    // FIXME: Save the different objects so they can be saved to files
    createGridFile(level);
    createCollisionsFile(level);
    for (Actor actor : level.getActors().getActorTypes()) {
      createActorFile(level, actor);
    }
    return null;
  }

  private JSONObject createActorFile(LevelIO level, Actor actor)
      throws NoSuchMethodException, IllegalAccessException, InvocationTargetException {
    JSONObject actorFile = new JSONObject();
    actorFile.put(ACTOR_KEY, actorWriter.writeActorObject(actor));
    actorFile.put(TRIGGERS_KEY, triggerWriter.writeTriggerObject(level.getTriggers(), actor));
    return actorFile;
  }

  private JSONArray createGridFile(LevelIO level) {
    return gridWriter.writeGridArray(level);
  }

  private JSONObject createCollisionsFile(LevelIO level) {
    return collisionWriter.writeCollisionObject(level.getCollisionManager());
  }
}
```

```java
package ooga.data.writers;

import ooga.engine.Coordinate;
import ooga.engine.actions.MovementManager;
import ooga.engine.actors.Actor;
import ooga.engine.property.Property;
import ooga.engine.property.PropertyList;
import org.json.JSONObject;

public class ActorWriter extends ReflectiveParser implements ReflectiveKeys {

  protected JSONObject writeActorObject(Actor actor) {
    JSONObject newActorData = new JSONObject();
    newActorData.put(CLASSNAME_KEY, actor.getClass().getName());
    newActorData.put(ID_KEY, actor.getID());
    newActorData.put(PROPERTIES_KEY, createActorProperties(actor.getProperties()));
    newActorData.put(DIMENSIONS_KEY, createActorDimensions(actor.getDimensions()));
    newActorData.put(MOVEMENT_KEY, createActorMovement(actor.getMovementManager()));
    return newActorData;
  }

  private JSONObject createActorProperties(PropertyList properties) {
    JSONObject actorProperties = new JSONObject();
    for (Object prop : properties.getViewableProperties()) {
      Property currentProperty = (Property) prop;
      actorProperties.put(currentProperty.getPropertyName(), currentProperty.isActive());
    }
    return actorProperties;
  }

  private JSONObject createActorMovement(MovementManager movementManager) {
    JSONObject actorMovementManager = new JSONObject();
    actorMovementManager.put(GRAVITY_KEY, movementManager.getGravity());
    actorMovementManager.put(FRICTION_KEY, movementManager.getFriction());
    actorMovementManager.put(MAXSPEED_KEY, movementManager.getMaxSpeed());
    actorMovementManager.put(TERMINAL_VELOCITY_KEY, movementManager.getTerminalVelocity());
    actorMovementManager
        .put(HORIZONTAL_ACCELERATION_KEY, movementManager.getHorizontalAcceleration());
    actorMovementManager.put(JUMP_FORCE_KEY, movementManager.getJumpForce());
    return actorMovementManager;
  }

  private JSONObject createActorDimensions(Coordinate dimensions) {
    JSONObject actorDimensions = new JSONObject();
    actorDimensions.put(WIDTH_KEY, dimensions.getX());
    actorDimensions.put(HEIGHT_KEY, dimensions.getY());
    return actorDimensions;
  }
}

```


### Impressions

This project was very enjoyable with a large amount of potential.  Seeing things materialize after many difficult discussions was very rewarding.  However, the project lasted much shorter than desired, as many more features could be planned and implemented if we had more time.  More time given would also allow us to clean up more code and abstract more code for increased openness for extension.

Fun project, especially when all the parsing was done and I could just make games. Really wish we had more time for this, considering we implemented a lot more features compared to SLogo yet both projects were given roughly the same amount of time. I would suggest saving more time for this final project, considering that it is worth half the final grade it should be at  a month and a half long if not more.

I had fun working with the big team on this project. However, I don't think I enjoyed the pure coding aspect of this project compared to others. I think part of the code I was writing did not have an immediate noticeable affect for anyone, including myself, so I wasn't as motivated with the work. With that being said, I believe my time spent on the project could have increased more in order for me to have roles that affect more of the code, but it was harder to find time as all my other courses were ending too. I also believe we had much more time than we did, so my higher expectations of the project turned sour, but that was because I failed to properly estimate time. 

version https://git-lfs.github.com/spec/v1
oid sha256:3a41b942d265decf5da04a185c42f076f08cba2e0d65834e8c47fa5eaa729761
size 11972
