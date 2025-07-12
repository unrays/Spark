# Crystal Engine

This project is a personal showcase developed at the end of my second year studying Computer Science Techniques at Cégep de Rimouski.  
It is a minimalist 2D game engine written in C++ using SDL2, built entirely from scratch.

The project is still a work in progress and currently contained in a single source file.  
It demonstrates my technical skills in object-oriented programming, input handling, graphics rendering, and software architecture.

This engine serves as a foundation for future improvements and projects I plan to develop.

```cpp
// Copyright (c) May 2025 Félix-Olivier Dumas. All rights reserved.
// Licensed under the terms described in the LICENSE file

#include <SDL.h>
#include <SDL_image.h>
#include <iostream>
#include <vector>
#include <cstdint>
#include <optional>
#include <cmath>
#include <tuple>

std::vector<SDL_Rect> sceneObjects;
SDL_Rect playerActor = { 25, 25, 50, 40 };
Uint8 playerColorR = 0, playerColorG = 255, playerColorB = 0;

class IPrintable {
public:
    virtual ~IPrintable() = default;
    virtual void print(std::ostream& stream) const = 0;
};

inline std::ostream& operator<<(std::ostream& stream, const IPrintable& obj) {
    obj.print(stream);
    return stream;
}

class IOperable { // Trop complexe à implémenter pour le moment
public:
    virtual ~IOperable() = default;
    virtual std::unique_ptr<IOperable> operator+(const IOperable& other) const = 0;
    virtual std::unique_ptr<IOperable> operator-(const IOperable& other) const = 0;
    virtual std::unique_ptr<IOperable> operator*(const IOperable& other) const = 0;
    virtual std::unique_ptr<IOperable> operator/(const IOperable& other) const = 0;

    virtual IOperable& operator+=(const IOperable& other) = 0;
    virtual IOperable& operator-=(const IOperable& other) = 0;
    virtual IOperable& operator*=(const IOperable& other) = 0;
    virtual IOperable& operator/=(const IOperable& other) = 0;
};

class IVector : public virtual IPrintable {
public:
    virtual ~IVector() = default;

    void print(std::ostream& stream) const = 0;

    virtual void setXY(const float x, const float y) = 0;

    virtual float getX() const = 0;
    virtual float getY() const = 0;

    virtual void setX(float x) = 0;
    virtual void setY(float y) = 0;
};

class Vector2 : public IVector {
private:
    float _x, _y;

public:
    Vector2(float x = 0.0f, float y = 0.0f) : _x(x), _y(y) {}

    void setXY(const float x, const float y) override { _x = x; _y = y; }

    void print(std::ostream& stream) const override { stream << "(" << _x << ", " << _y << ")" << std::endl; }

    float getX() const override { return _x; }
    void setX(float x) override { _x = x; }

    float getY() const override { return _y; }
    void setY(float y) override { _y = y; }

    Vector2 operator+(const Vector2& other) const { return Vector2(_x + other.getX(), _y + other.getY()); }
    Vector2 operator-(const Vector2& other) const { return Vector2(_x - other.getX(), _y - other.getY()); }
    Vector2 operator*(const Vector2& other) const { return Vector2(_x * other.getX(), _y * other.getY()); }
    Vector2 operator/(const Vector2& other) const { return Vector2(_x / other.getX(), _y / other.getY()); }

    Vector2& operator+=(const Vector2& other) { this->setXY(this->_x += other.getX(), this->_y += other.getY()); return *this; }
    Vector2& operator-=(const Vector2& other) { this->setXY(this->_x -= other.getX(), this->_y -= other.getY()); return *this; }
    Vector2& operator*=(const Vector2& other) { this->setXY(this->_x *= other.getX(), this->_y *= other.getY()); return *this; }
    Vector2& operator/=(const Vector2& other) { this->setXY(this->_x /= other.getX(), this->_y /= other.getY()); return *this; }
};

class IColor : public virtual IPrintable {
public:
    virtual ~IColor() = default;

    virtual Uint8 getR() const = 0;
    virtual Uint8 getG() const = 0;
    virtual Uint8 getB() const = 0;
    virtual Uint8 getA() const = 0;

    virtual void setColor(Uint8 r, Uint8 g, Uint8 b, Uint8 a = 255) = 0;

    virtual void print(std::ostream& stream) const = 0;
};

class ColorRGBA : public IColor {
private:
    Uint8 _r, _g, _b, _a;

public:
    static constexpr Uint8 MAX_COLOR_VALUE = 255;
    static constexpr Uint8 MIN_COLOR_VALUE = 0;

    ColorRGBA(Uint8 r = 0, Uint8 g = 0, Uint8 b = 0, Uint8 a = 255)
        : _r(r), _g(g), _b(b), _a(a) {}

    void setColor(Uint8 r, Uint8 g, Uint8 b, Uint8 a) { _r = r; _g = g; _b = b; _a = a; }

    Uint8 getR() const { return _r; }
    Uint8 getG() const { return _g; }
    Uint8 getB() const { return _b; }
    Uint8 getA() const { return _a; }

    Uint8 setR(Uint8 r) { _r = r; }
    Uint8 setG(Uint8 g) { _g = g; }
    Uint8 setB(Uint8 b) { _b = b; }
    Uint8 setA(Uint8 a) { _a = a; }

    void print(std::ostream& stream) const override { stream << "(" << _r << ", " << _g << ", " << _b << ", " << _a << ")"; }

    ColorRGBA operator+(const ColorRGBA& other) const {
        return ColorRGBA((_r + other.getR() > MAX_COLOR_VALUE) ? MAX_COLOR_VALUE : _r + other.getR(),
                         (_g + other.getG() > MAX_COLOR_VALUE) ? MAX_COLOR_VALUE : _g + other.getG(),
                         (_b + other.getB() > MAX_COLOR_VALUE) ? MAX_COLOR_VALUE : _b + other.getB(),
                         (_a + other.getA() > MAX_COLOR_VALUE) ? MAX_COLOR_VALUE : _a + other.getA());
    }

    ColorRGBA operator-(const ColorRGBA& other) const {
        return ColorRGBA((_r - other.getR() < MIN_COLOR_VALUE) ? MIN_COLOR_VALUE : _r - other.getR(),
                         (_g - other.getG() < MIN_COLOR_VALUE) ? MIN_COLOR_VALUE : _g - other.getG(),
                         (_b - other.getB() < MIN_COLOR_VALUE) ? MIN_COLOR_VALUE : _b - other.getB(),
                         (_a - other.getA() < MIN_COLOR_VALUE) ? MIN_COLOR_VALUE : _a - other.getA());
    }

    ColorRGBA& operator+=(const ColorRGBA& other) {
        this->setColor((_r + other.getR() > MAX_COLOR_VALUE) ? MAX_COLOR_VALUE : _r + other.getR(),
                       (_g + other.getG() > MAX_COLOR_VALUE) ? MAX_COLOR_VALUE : _g + other.getG(),
                       (_b + other.getB() > MAX_COLOR_VALUE) ? MAX_COLOR_VALUE : _b + other.getB(),
                       (_a + other.getA() > MAX_COLOR_VALUE) ? MAX_COLOR_VALUE : _a + other.getA());
        return *this;
    }

    ColorRGBA& operator-=(const ColorRGBA& other) {
        this->setColor((_r - other.getR() < MIN_COLOR_VALUE) ? MIN_COLOR_VALUE : _r - other.getR(),
                       (_g - other.getG() < MIN_COLOR_VALUE) ? MIN_COLOR_VALUE : _g - other.getG(),
                       (_b - other.getB() < MIN_COLOR_VALUE) ? MIN_COLOR_VALUE : _b - other.getB(),
                       (_a - other.getA() < MIN_COLOR_VALUE) ? MIN_COLOR_VALUE : _a - other.getA());
        return *this;
    }
};

class ISizable {
public:
    virtual ~ISizable() = default;
    virtual void scale(const float& factor) = 0;
};

class Size : public ISizable {
private:
    float _width, _height;

public:
    ~Size() = default;
    Size(const float width, const float height)
        : _width(width), _height(height) {}

    float getWidth()  const { return _width; }
    float getHeight() const { return _height; }

    void scale(const float& factor) override {
        _width *= factor; _height *= factor;
    }
};

class IMovable {
public:
    virtual ~IMovable() = default;
    virtual void move(const Vector2& offset) = 0;
    virtual void setPosition(const Vector2& position) = 0;
    virtual Vector2 currentPosition() const = 0;
};

class IRotatable {
public:
    virtual ~IRotatable() = default;
    virtual void setRotation(const float& angle) = 0;
    virtual void rotate(const float& angle) = 0;
};

class Position : public virtual IMovable, public virtual IPrintable {
private:
    Vector2 _coordinates;

public:
    Position() = default;
    Position(const Vector2& coords) : _coordinates(coords) { }

    float getX() const { return _coordinates.getX(); }
    float getY() const { return _coordinates.getY(); }

    void setX(float x) { _coordinates.setX(x); }
    void setY(float y) { _coordinates.setX(y); }

    void setPosition(const Vector2& newPosition) override { _coordinates = newPosition; }
    Vector2 currentPosition() const override { return _coordinates; }

    void move(const Vector2& offset) override { _coordinates.setXY(_coordinates.getX() + offset.getX(), _coordinates.getY() + offset.getY()); }

    void print(std::ostream& stream) const override { stream << _coordinates; };

    Position operator+(const Position& other) const { return Position(_coordinates + other.currentPosition()); }
    Position operator-(const Position& other) const { return Position(_coordinates - other.currentPosition()); }
    Position operator*(const Position& other) const { return Position(_coordinates * other.currentPosition()); }
    Position operator/(const Position& other) const { return Position(_coordinates / other.currentPosition()); }

    Position& operator+=(const Position& other) { _coordinates += other.currentPosition(); return *this; }
    Position& operator-=(const Position& other) { _coordinates -= other.currentPosition(); return *this; }
    Position& operator*=(const Position& other) { _coordinates *= other.currentPosition(); return *this; }
    Position& operator/=(const Position& other) { _coordinates /= other.currentPosition(); return *this; }
};

class Rotation : public virtual IRotatable, public virtual IPrintable {
private:
    float _angle;

public:
    static constexpr float MAX_ANGLE = 360.0f;
    static constexpr float MIN_ANGLE = 360.0f;

    Rotation(const float& angle) : _angle(angle) { }

    void setRotation(const float& newAngle) override { _angle = (newAngle < MAX_ANGLE) ? newAngle : 0.0f; }

    void print(std::ostream& stream) const override { stream << _angle; };

    void rotate(const float& angle) override {
        const float range = MAX_ANGLE - MIN_ANGLE;

        _angle += angle;
        _angle = std::fmod(_angle - MIN_ANGLE, range);

        if (_angle < 0) {
            _angle += range;
            _angle += MIN_ANGLE;
        }
    }
};

class ICollidable {
public:
    virtual ~ICollidable() = default;
    virtual void collideWith(const ICollidable& object) = 0;
    virtual const Position& getHitboxPosition() const = 0;
};

class IPhysicsApplicable {
public:
    virtual ~IPhysicsApplicable() = default;
};

class IRenderable {
public:
    virtual void render(SDL_Renderer* renderer) = 0;
    virtual ~IRenderable() = default;
};

class ISpriteable {
public:
    virtual ~ISpriteable() = default;
    virtual void changeSprite(SDL_Renderer* renderer, const std::string& imagePath) = 0;
};

class IAnimatable {
public:
    virtual ~IAnimatable() = default;
    virtual void changeAnimation(SDL_Renderer* renderer, const Vector2& resolution, const Vector2& gridSize, const std::string& imagePath) = 0;
    virtual void setAnimationFrame(const int row, const int column) = 0;
};

class ITexturable {
public:
    virtual ~ITexturable() = default;
    virtual void loadFromFile(const std::string& imagePath) = 0;
    virtual void convertToTexture(SDL_Renderer* renderer) = 0;
};

class IControllable {
public:
    virtual ~IControllable() = default;
    virtual void handleInput(SDL_Renderer* renderer, const SDL_Event& input) = 0;
};

class ITargetable : public virtual IMovable {
public:
    virtual ~ITargetable() = default;
};

class IHealthable {
public:
    virtual ~IHealthable() = default;
    virtual void heal(const Uint32 healAmount) = 0;
    virtual void takeDamage(const Uint32 damageAmount) = 0;
    virtual bool isAlive() const = 0;
    virtual Uint32 getHealth() const = 0;
};

struct IZoomable {
public:
    virtual ~IZoomable() = default;
    virtual void zoom(const float& factor) = 0;
};

class Camera : public virtual IMovable, public virtual IRotatable, public IZoomable {
private:
    std::optional<ITargetable*> _target;
    Position _position;
    Rotation _angle;

    bool _isTargeting = false;
    Position _displacement;

    Vector2 _lastPosition;

public:
    Camera(Position& position, Rotation& angle, ITargetable* target = nullptr)
        : _position(position), _angle(angle) {
        if (target) { _target = target; _isTargeting = true; }
    }

    void setTarget(ITargetable& target) { _target = &target; _isTargeting = true; }
    void clearTarget() { _target.reset(); _isTargeting = false; }

    void update() {
        if (!_target) {
            _target.emplace();
            std::cerr << "[WARNING] No initialized target for update\n";
            return;
        }
        _lastPosition = _position.currentPosition();

        Vector2 newPos = _target.value()->currentPosition() + _displacement.currentPosition();
        _position.setPosition(newPos);

        _displacement.setPosition(Vector2(0.f, 0.f));
    }


    void setPosition(const Vector2& position) override { _position.setPosition(position); }
    Vector2 currentPosition() const override { return _position.currentPosition(); }
    Vector2 lastPosition() const { return _lastPosition; }

    void move(const Vector2& offset) override { _displacement += offset; }


    void setRotation(const float& angle) override { _angle.setRotation(angle); }
    void rotate(const float& angle) override { _angle.rotate(angle); }

    void zoom(const float& factor) override { /* [À FINIR] */ }
};

class Sprite : public virtual ITexturable {
private:
    SDL_Surface* _surface = nullptr;
    SDL_Texture* _texture = nullptr;
    std::string  _imagePath;

public:
    Sprite() = default;
    Sprite(const std::string& imagePath) : _imagePath(imagePath) { }

    void loadFromFile(const std::string& imagePath = "") override {
        const std::string loadedPath = (imagePath.empty() ? _imagePath : imagePath);
        _surface = IMG_Load(loadedPath.c_str());

        if (!_surface) { std::cerr << "[ERROR] Can't load the surface: " << IMG_GetError() << std::endl; }
    }

    void convertToTexture(SDL_Renderer* renderer) override {
        _texture = SDL_CreateTextureFromSurface(renderer, _surface);
        if (!_texture) { std::cout << "[ERROR] SDL_CreateTextureFromSurface: " << SDL_GetError() << std::endl; }
        else { SDL_FreeSurface(_surface); _surface = nullptr; }
    }

    void draw(SDL_Renderer* renderer, const Position& position, const Size& size) {
        SDL_Rect destination = { static_cast<int>(position.getX()), static_cast<int>(position.getY()),
                                 static_cast<int>(size.getWidth()), static_cast<int>(size.getHeight()) };
        SDL_RenderCopy(renderer, _texture, NULL, &destination);
    }
};

class SpriteSheet : public virtual ITexturable {
private:
    SDL_Rect     m_clip;
    SDL_Surface* _surface = nullptr;
    SDL_Texture* _texture = nullptr;
    std::string  _imagePath;
    int _spriteWidth = 0;
    int _spriteHeight = 0;
    int _padding = 0;
    int _columns = 0;
    int _rows = 0;

public:
    SpriteSheet() = default;
    SpriteSheet(const std::string& imagePath) : _imagePath(imagePath) { }

    void setSpriteSheetDimensions(int sheetWidth, int sheetHeight, int rows, int columns, int padding = 0) {
        _rows = rows;
        _columns = columns;
        _padding = padding;
        _spriteWidth = (sheetWidth - (columns - 1) * padding) / columns;
        _spriteHeight = (sheetHeight - (rows - 1) * padding) / rows;
        m_clip = { 0, 0, _spriteWidth, _spriteHeight };
    }

    void loadFromFile(const std::string& imagePath = "") override {
        const std::string loadedPath = imagePath.empty() ? _imagePath : imagePath;
        _surface = IMG_Load(loadedPath.c_str());
        if (!_surface) {
            std::cerr << "[ERROR] Can't load the surface: " << IMG_GetError() << std::endl;
        }
    }

    void convertToTexture(SDL_Renderer* renderer) override {
        _texture = SDL_CreateTextureFromSurface(renderer, _surface);
        if (!_texture) {
            std::cerr << "[ERROR] SDL_CreateTextureFromSurface: " << SDL_GetError() << std::endl;
        }
        else {
            SDL_FreeSurface(_surface);
            _surface = nullptr;
        }
    }

    void selectSprite(int x, int y) {
        m_clip.x = x * (_spriteWidth + _padding);
        m_clip.y = y * (_spriteHeight + _padding);
    }

    void selectSpriteByTime(int fps = 10, int frameCount = 1, int row = 0) {
        Uint32 ticks = SDL_GetTicks();
        Uint32 spriteIndex = (ticks / (1000 / fps)) % frameCount;

        int x = spriteIndex % _columns;
        int y = row;

        selectSprite(x, y);
    }



    void draw(SDL_Renderer* renderer, const Position& position, const Size& size) {
        if (!_texture) {
            std::cerr << "[ERROR] No texture loaded, impossible to draw." << std::endl;
            return;
        }

        SDL_Rect destRect = {
            static_cast<int>(position.getX()),
            static_cast<int>(position.getY()),
            static_cast<int>(size.getWidth()),
            static_cast<int>(size.getHeight())
        };
        SDL_RenderCopy(renderer, _texture, &m_clip, &destRect);
    }
};



class GameObject : public IRenderable, public ISizable, public virtual IMovable {
protected:
    Position _position;
    Size _size;

public:
    GameObject(Position& position, const Size& size) 
        : _position(position), _size(size) { }

    virtual ~GameObject() = default;

    void scale(const float& factor) { _size.scale(factor); }

    void setPosition(const Vector2& position) override { _position.setPosition(position); }
    Vector2 currentPosition() const override { return _position.currentPosition(); }

    void move(const Vector2& offset) override { _position.move(offset); }
};

class Actor : public GameObject, public ISpriteable, public IAnimatable, public ITargetable {
protected:
    std::optional<Sprite> _sprite;
    std::optional<SpriteSheet> _animation;

public:
    Actor(Position& position, const Size& size, std::optional<Sprite> sprite = std::nullopt, std::optional<SpriteSheet> animation = std::nullopt)
        : GameObject(position, size), _sprite(std::move(sprite)), _animation(std::move(animation)) { }

    void render(SDL_Renderer* renderer) override {
        if (_sprite) { _sprite->draw(renderer, _position, _size); }
        else if (_animation) { _animation->draw(renderer, _position, _size); }
    }

    void changeSprite(SDL_Renderer* renderer, const std::string& imagePath) override {
        if (!_sprite) _sprite.emplace();

        _sprite->loadFromFile(imagePath);
        _sprite->convertToTexture(renderer);
    }

    void changeAnimation(SDL_Renderer* renderer, const Vector2& resolution, const Vector2& gridSize, const std::string& imagePath) override {
        if (!_animation) _animation.emplace();

        _animation->loadFromFile(imagePath);
        _animation->convertToTexture(renderer);
        _animation->setSpriteSheetDimensions(static_cast<int>(resolution.getX()), static_cast<int>(resolution.getY()),
                                             static_cast<int>(gridSize.getX()), static_cast<int>(gridSize.getY()));
    }

    void setAnimationFrame(const int row, const int column) override {
        if (!_animation) _animation.emplace();

        _animation->selectSprite(row, column);
    }

    void testAnim() {
        if (!_animation) _animation.emplace();

        _animation->selectSpriteByTime(4, 8, 1);
    }
};

class Player : public virtual Actor, public virtual IHealthable, public IControllable, public virtual ICollidable {
private:
    Uint32 _health; Uint32 _maxHealth;

public:
    Player(Position& position, const Size& size, Uint32 health = 100, Uint32 maxHealth = 100,
        std::optional<Sprite> sprite = std::nullopt, std::optional<SpriteSheet> spriteSheet = std::nullopt)
        : Actor(position, size, sprite, spriteSheet), _health(health), _maxHealth(maxHealth) {}

    void heal(const Uint32 healAmount) override { _health = (_health + healAmount <= _maxHealth) ? (_health + healAmount) : _maxHealth; }
    void takeDamage(const Uint32 damageAmount) override { _health -= damageAmount; }
    bool isAlive() const override { return _health > 0;  }
    Uint32 getHealth() const override { return _health; }

    void collideWith(const ICollidable& object) override {
        /* Detecter les collisions hors d'ici
           Et gerer les effets de cette collision
           Dans cette methode (genre ne pas permettre
           De traverser l'objet)... Peut-être penser
           à faire une classe abstraite quelque part pour
           move, scale et tout car ils sont communs [FAIT] */
    }

    const Position& getHitboxPosition() const override { return _position; }

    void handleInput(SDL_Renderer* renderer, const SDL_Event& event) override {
        if (event.type == SDL_KEYDOWN) {
            if (event.key.keysym.sym == SDLK_SPACE) {
                std::cout << "Espace pressée !" << std::endl;
            }
            else if (event.key.keysym.sym == SDLK_w) {
                //move(Vector2(0, -15));
                //changeSprite(renderer, "assets/images/mario_jump_sprite.png");
            }
            else if (event.key.keysym.sym == SDLK_s) {
                //move(Vector2(0, 15));
            }
            else if (event.key.keysym.sym == SDLK_a) {
                //move(Vector2(-15, 0));
            }
            else if (event.key.keysym.sym == SDLK_d) {
                //move(Vector2(15, 0));
                //_animation->selectSpriteByTime(10, 8, 4);
            }
            else {
                testAnim();
            }

        }
    }
};

class Enemy : public virtual Actor, public virtual IHealthable, public virtual ICollidable {
private:
    Uint32 _health; Uint32 _maxHealth;

public:
    Enemy(Position& position, const Size& size, Uint32 health = 100, Uint32 maxHealth = 100,
        std::optional<Sprite> sprite = std::nullopt, std::optional<SpriteSheet> spriteSheet = std::nullopt)
        : Actor(position, size, sprite, spriteSheet), _health(health), _maxHealth(maxHealth) {}

    void heal(const Uint32 healAmount) override { _health = (_health + healAmount <= _maxHealth) ? (_health + healAmount) : _maxHealth; }
    void takeDamage(const Uint32 damageAmount) override { _health -= damageAmount; }
    bool isAlive() const override { return _health > 0; }
    Uint32 getHealth() const override { return _health; }

    const Position& getHitboxPosition() const override { return _position; }

    void collideWith(const ICollidable& object) override {
    }
};

struct configuration {
    int width;
    int height;
};

void handleKeyboardInput(SDL_Renderer* renderer, SDL_Event& event, Player& player, bool& isRunning) { // [USELESS]
    switch (event.type) {
    case SDL_QUIT:
        isRunning = false;
        break;

    case SDL_KEYDOWN:
        if (SDL_GetScancodeName(event.key.keysym.scancode) == std::string("W")) {
            player.move(Vector2(0, -15));
        }
        if (SDL_GetScancodeName(event.key.keysym.scancode) == std::string("S")) {
            player.move(Vector2(0, 15));
        }
        if (SDL_GetScancodeName(event.key.keysym.scancode) == std::string("A")) {
            player.move(Vector2(-15, 0));
        }
        if (SDL_GetScancodeName(event.key.keysym.scancode) == std::string("D")) {
            player.move(Vector2(15, 0));
        }

        std::cout << "[KEYBOARD INPUT] Scancode: " << SDL_GetScancodeName(event.key.keysym.scancode) << std::endl;
        break;


    default:
        break;
    }
}

float stickX = 0.0f;
float stickY = 0.0f;

void handleControllerInput(SDL_Renderer* renderer, SDL_Event &event, Player& player, bool &isRunning) { // [USELESS]
    switch (event.type) {
    case SDL_QUIT:
        isRunning = false;
        break;

    case SDL_CONTROLLERBUTTONDOWN:
        if (event.cbutton.button == SDL_CONTROLLER_BUTTON_A) {
            std::cout << "[CONTROLLER] A PRESSED" << std::endl;

            playerColorR = 255;


        }
        if (event.cbutton.button == SDL_CONTROLLER_BUTTON_B) {
            std::cout << "[CONTROLLER] B PRESSED" << std::endl;
        }
        if (event.cbutton.button == SDL_CONTROLLER_BUTTON_X) {
            std::cout << "[CONTROLLER] X PRESSED" << std::endl;
        }
        if (event.cbutton.button == SDL_CONTROLLER_BUTTON_Y) {
            std::cout << "[CONTROLLER] Y PRESSED" << std::endl;
        }

        if (event.cbutton.button == SDL_CONTROLLER_BUTTON_DPAD_UP) {
            std::cout << "[CONTROLLER] DPAD UP PRESSED" << std::endl;
            //playerActor.y -= 15;

            player.move(Vector2(0, -5));
        }
        if (event.cbutton.button == SDL_CONTROLLER_BUTTON_DPAD_DOWN) {
            std::cout << "[CONTROLLER] DPAD DOWN PRESSED" << std::endl;
            //playerActor.y += 15;

            player.move(Vector2(0, 5));
        }
        if (event.cbutton.button == SDL_CONTROLLER_BUTTON_DPAD_LEFT) {
            std::cout << "[CONTROLLER] DPAD LEFT PRESSED" << std::endl;
            //playerActor.x -= 15;

            player.move(Vector2(-5, 0));
        }
        if (event.cbutton.button == SDL_CONTROLLER_BUTTON_DPAD_RIGHT) {
            std::cout << "[CONTROLLER] DPAD RIGHT PRESSED" << std::endl;
            //playerActor.x += 15;

            player.move(Vector2(5, 0));
        }

        if (event.cbutton.button == SDL_CONTROLLER_BUTTON_RIGHTSHOULDER) {
            std::cout << "[CONTROLLER] RB PRESSED" << std::endl;
        }
        if (event.cbutton.button == SDL_CONTROLLER_BUTTON_LEFTSHOULDER) {
            std::cout << "[CONTROLLER] LB PRESSED" << std::endl;
        }

        if (event.cbutton.button == SDL_CONTROLLERAXISMOTION) {

        }
        break;

    case SDL_CONTROLLERAXISMOTION:
        if (event.caxis.axis == SDL_CONTROLLER_AXIS_TRIGGERRIGHT) {
            if (event.caxis.value == 32767) {
                std::cout << "[CONTROLLER] RT PRESSED: " << std::endl;
            }
        }
        if (event.caxis.axis == SDL_CONTROLLER_AXIS_TRIGGERLEFT) {
            if (event.caxis.value == 32767) {
                std::cout << "[CONTROLLER] LT PRESSED: " << std::endl;
            }
        }

        

        if (event.caxis.axis == SDL_CONTROLLER_AXIS_LEFTX) {
            stickX = event.caxis.value;
            player.move(Vector2((stickX / 10000), 0));
            
            
           
        }
        if (event.caxis.axis == SDL_CONTROLLER_AXIS_LEFTY) {
            stickY = event.caxis.value;
            player.move(Vector2(0, (stickY / 10000)));


        }
        break;

    default:
        break;
    }
}

int main(int argc, char* argv[]) {
    if (SDL_Init(SDL_INIT_VIDEO | SDL_INIT_GAMECONTROLLER) != 0) {
        SDL_Log("Erreur SDL_Init : %s", SDL_GetError());
        return 1;
    }

    SDL_GameController* controller = nullptr;
    if (SDL_NumJoysticks() > 0 && SDL_IsGameController(0)) {
        controller = SDL_GameControllerOpen(0);
        std::cout << ((controller ? "Controller detected and open" : "No controller detected or failed to open")) << std::endl;
    }

    SDL_Window* window = SDL_CreateWindow("SDL2 Setup Simple",
        SDL_WINDOWPOS_CENTERED,
        SDL_WINDOWPOS_CENTERED,
        800, 600,
        SDL_WINDOW_SHOWN);

    if (!window) {
        SDL_Log("Erreur SDL_CreateWindow : %s", SDL_GetError());
        SDL_Quit();
        return 1;
    }

    SDL_Renderer* renderer = SDL_CreateRenderer(window, -1, SDL_RENDERER_ACCELERATED);
    if (!renderer) {
        SDL_Log("Erreur SDL_CreateRenderer : %s", SDL_GetError());
        SDL_DestroyWindow(window);
        SDL_Quit();
        return 1;
    }

    bool running = true;
    SDL_Event event;

    configuration configuration;
    SDL_GL_GetDrawableSize(window, &configuration.width, &configuration.height);
    std::cout << "" << configuration.width << std::endl << configuration.height << std::endl;

    static float offset = 0;
    //SDL_Rect rect = { 100, 100, 200, 150 };
    //sceneObjects.push_back(rect);
    sceneObjects.push_back(playerActor);

    /*********************************************************/

    SpriteSheet playerSpriteSheet("assets/images/sprite_sheet_new.png");

    playerSpriteSheet.loadFromFile();
    playerSpriteSheet.setSpriteSheetDimensions(1200, 510, 6, 8);
    playerSpriteSheet.convertToTexture(renderer);

    playerSpriteSheet.selectSprite(0, 5);


    /*********************************************************/

    Sprite playerSprite("");

    playerSprite.loadFromFile("assets/images/mario_sprite.png");
    playerSprite.convertToTexture(renderer);

    //playerSprite.testImageLoad("C:/Users/XXXXXXXXXX/Documents/Programmation/Crystal Engine/assets/images/sprite.png", renderer);

    Position playerPosition(Vector2(-200.0f, -100.0f));
    Size playerSize(775.0f, 600.0f);

    //Player player(std::move(playerPosition), playerSize, 100, 100, playerSprite, std::nullopt);
    Player player(playerPosition, playerSize, 100, 100, std::nullopt, playerSpriteSheet);


    Sprite fActorSprite("");

    fActorSprite.loadFromFile("assets/images/mario_sprite.png");
    fActorSprite.convertToTexture(renderer);

    Position fActorPosition(Vector2(static_cast<float>(configuration.width / 2), 50.0f));
    Size fActorSize(500.0f, 500.0f);

    Enemy fActor(fActorPosition, fActorSize, 100, 100, fActorSprite, std::nullopt);

    /*********************************************************/

    Position cameraStart(Vector2(0, 0));
    Rotation cameraAngle(0.0f);

    Camera camera(cameraStart, cameraAngle, &player);

    //camera.move(Vector2(10, 0));

    /*********************************************************/

    std::vector<GameObject*> renderQueue;
    //renderQueue.push_back(&player);
    renderQueue.push_back(&fActor);

    static int sprite = 0;

    while (running) {
        //rect.x = (configuration.width / 2) - (rect.w / 2);
        //rect.y = (configuration.height / 2) - (rect.h / 2);

        const Position& playerHitBox = player.getHitboxPosition();

        /*if (std::round(rect.x) == std::round(playerPositionX)) {
            std::cout << "Collision Detected" << std::endl;
        }

        if (playerPositionX >= (rect.x) && playerPositionX <= (rect.x + rect.w) &&
            playerPositionY >= (rect.y) && playerPositionY <= (rect.y + rect.h)) {
            std::cout << "Le point est dans la zone !" << std::endl;
        }*/

        //std::cout << playerPositionX << ", " << playerPositionY << std::endl;
        //std::cout << rect.x << ", " << rect.y << std::endl;

        while (SDL_PollEvent(&event)) {
            if (event.type == SDL_QUIT) running = false;

            handleKeyboardInput(renderer, event, player, running);
            handleControllerInput(renderer, event, player, running);

            player.handleInput(renderer, event);
        }
        
        SDL_SetRenderDrawColor(renderer, 0, 0, 0, 255);
        SDL_RenderClear(renderer);

        camera.update();

        Vector2 cameraPos = camera.currentPosition();
        Vector2 screenCenter(configuration.width / 2.f, configuration.height / 2.f);

        for (GameObject* object : renderQueue) {
            Vector2 worldPos = object->currentPosition();
            Vector2 screenPos = worldPos - cameraPos + screenCenter;


            object->setPosition(worldPos);
            object->render(renderer);

            std::cout << worldPos << std::endl;
        }




        //playerSpriteSheet.selectSprite(sprite, 0);

        //player.setAnimationFrame(sprite, 0);
        player.testAnim();

        player.render(renderer);
        fActor.render(renderer);

        camera.update();

        Vector2 camPos = camera.currentPosition();
        std::cout << "Camera position: (" << camPos.getX() << ", " << camPos.getY() << ")\n";
        
        //player.updateJump(1.0f / 60.0f);


        SDL_SetRenderDrawColor(renderer, 255, 0, 0, 255);
        //SDL_RenderFillRect(renderer, &rect);

        SDL_SetRenderDrawColor(renderer, playerColorR, playerColorG, playerColorB, 255);
        SDL_RenderFillRect(renderer, &playerActor); // remplacer

        SDL_RenderPresent(renderer);

        sprite++;

        SDL_Delay(16);
    }

    if (controller) SDL_GameControllerClose(controller);

    SDL_DestroyRenderer(renderer);
    SDL_DestroyWindow(window);
    SDL_Quit();
    return 0;
}
```