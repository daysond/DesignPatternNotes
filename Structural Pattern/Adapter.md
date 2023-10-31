# Adapter

**Adapter** is a **structural** design pattern that allows objects with incompatible interfaces to collaborate.

## Reference

Material: [Adapter ](https://refactoring.guru/design-patterns/Adapter)

Sample & Complex Code: [SED505 - Design Pattern](https://github-pages.senecacollege.ca/sed505/#) 

## Why?

- Want to make two incompatible interfaces of different classes to work together 
    - integrating legacy code with new code
    - working with third-party libraries or APIs that have different interfaces

## How

- creating an adapter class interfacing the two incompatible interfaces
    - the adapter gets an interface, compatible with one of the existing objects
    - the existing object can safely call the adapter's methods using this interface
    - the adapter passes request to the second objects in a format that the second object expects

## Structure

### Object adapter

This implementation uses the object composition principle: the adapter implements the interface of one object and wraps the other one. It can be implemented in all popular programming languages.

<img src="https://refactoring.guru/images/patterns/diagrams/adapter/structure-object-adapter-2x.png" width=600px>

1. The **Client** is a class that contains the existing business logic of the program.

2. The **Client Interface** describes a protocol that other classes must follow to be able to collaborate with the client code.

3. The **Service** is some useful class (usually 3rd-party or legacy). The client can’t use this class directly because it has an incompatible interface.

4. The **Adapter** is a class that’s able to work with both the client and the service: it implements the client interface, while wrapping the service object. The adapter receives calls from the client via the client interface and translates them into calls to the wrapped service object in a format it can understand.

5. The client code doesn’t get coupled to the concrete adapter class as long as it works with the adapter via the client interface. Thanks to this, you can introduce new types of adapters into the program without breaking the existing client code. This can be useful when the interface of the service class gets changed or replaced: you can just create a new adapter class without changing the client code.

### Class adapter

This implementation uses inheritance: the adapter inherits interfaces from both objects at the same time. Note that this approach can only be implemented in programming languages that support multiple inheritance, such as C++.

<img src="https://refactoring.guru/images/patterns/diagrams/adapter/structure-class-adapter-2x.png" width=600px>

The **Class Adapter** doesn’t need to wrap any objects because it inherits behaviors from both the client and the service. The adaptation happens within the overridden methods. The resulting adapter can be used in place of an existing client class.



## Sample Code

```cpp
#include <iostream>

// Target interface
class ITarget
{
public:
    virtual void request() = 0;
};

// Adaptee class with incompatible interface (ie the legacy code)
class Adaptee
{
public:
    void specificRequest()
    {
        std::cout << "Adaptee specific request." << std::endl;
    }
};

// Adapter class (ie the modern code)
class Adapter : public ITarget
{
private:
    Adaptee* adaptee;

public:
    Adapter(Adaptee* adaptee)
    {
        this->adaptee = adaptee;
    }

    void request() override//(ie from the modern code to the legacy code)
    {
        adaptee->specificRequest();
    }
};

// Client code
void client(ITarget* target)
{
    target->request();
}

int main()
{
    Adaptee* adaptee = new Adaptee;
    ITarget* target = new Adapter(adaptee);
    client(target);

    delete adaptee;
    delete target;
    return 0;
}
```

## Complex Code - Legacy Media Player

### MediaPlayer.h

```cpp
#include <iostream>

class MediaPlayer {
public:
    virtual void play(std::string audioType, std::string fileName) = 0;
};
```

### MediaAdapter.h

```cpp
#include "Mp4Player.h"
#include "VlcPlayer.h"

class MediaAdapter : public MediaPlayer {
private:
    AdvancedMediaPlayer* advancedMediaPlayer;

public:
    MediaAdapter(std::string audioType) {
        if (audioType == "vlc")
            advancedMediaPlayer = new VlcPlayer();
        else if (audioType == "mp4")
            advancedMediaPlayer = new Mp4Player();
        else
            throw std::invalid_argument("Unsupported audio type.");//potential memory leak? Yes.
                                                                   //MediaAdapter in the main funciton is not deleted.
    }

    void play(std::string audioType, std::string fileName) override {
        if (audioType == "vlc")
            advancedMediaPlayer->playVlc(fileName);
        else if (audioType == "mp4")
            advancedMediaPlayer->playMp4(fileName);
    }
};

```

### AdvancedMediaPlayer.h

```cpp
#include "MediaPlayer.h"

class AdvancedMediaPlayer {
public:
    virtual void playVlc(std::string fileName) = 0;
    virtual void playMp4(std::string fileName) = 0;
};
```

### Mp4Player.h

```cpp
#include "AdvancedMediaPlayer.h"

class Mp4Player : public AdvancedMediaPlayer {
public:
    void playVlc(std::string fileName) override {
        // Do nothing
    }

    void playMp4(std::string fileName) override {
        std::cout << "Playing MP4 file: " << fileName << std::endl;
    }
};

```

### VlcPlayer.h

```cpp
#include "AdvancedMediaPlayer.h"

class VlcPlayer : public AdvancedMediaPlayer {
public:
    void playVlc(std::string fileName) override {
        std::cout << "Playing VLC file: " << fileName << std::endl;
    }

    void playMp4(std::string fileName) override {
        // Do nothing
    }
};

```

### MediaPlayerMain.cpp
 
```cpp
#include "MediaAdapter.h"
#include "MediaPlayer.h"

// Client
int main() {
    //adapt play (new code) to playVlc and playMp4 (old code)
    MediaPlayer* mediaPlayer = new MediaAdapter("mp4");
    mediaPlayer->play("mp4", "movie.mp4");
    delete mediaPlayer;

    mediaPlayer = new MediaAdapter("vlc");
    mediaPlayer->play("vlc", "song.vlc");
    delete mediaPlayer;

    //Code for a .wav player does not exist
    mediaPlayer = new MediaAdapter("wav");
    mediaPlayer->play("wav", "song.wav");
    delete mediaPlayer;

    return 0;
}
```