#include <SFML/Graphics.hpp>
#include <SFML/Audio.hpp>
#include <vector>
#include <cstdlib>
#include <ctime>

const int WIDTH = 800;
const int HEIGHT = 600;
const float GRAVITY = 0.4f;
const float JUMP_STRENGTH = -8.0f;
const int PIPE_WIDTH = 80;
const int PIPE_GAP = 200;
const float PIPE_SPEED = 3.0f;

struct Pipe {
    sf::RectangleShape upper;
    sf::RectangleShape lower;
    float x;
};

int main() {
    sf::RenderWindow window(sf::VideoMode(WIDTH, HEIGHT), "Flappy Bird");
    sf::Texture birdTexture;
    birdTexture.loadFromFile("bird.png");
    sf::Sprite bird(birdTexture);
    bird.setPosition(100, HEIGHT / 2);

    float velocityY = 0;
    std::vector<Pipe> pipes;
    srand(time(0));

    // Carregar sons
    sf::SoundBuffer jumpBuffer, pointBuffer, hitBuffer;
    jumpBuffer.loadFromFile("jump.wav");
    pointBuffer.loadFromFile("point.wav");
    hitBuffer.loadFromFile("hit.wav");
    
    sf::Sound jumpSound(jumpBuffer);
    sf::Sound pointSound(pointBuffer);
    sf::Sound hitSound(hitBuffer);
    
    sf::Clock clock;
    int score = 0;

    while (window.isOpen()) {
        sf::Event event;
        while (window.pollEvent(event)) {
            if (event.type == sf::Event::Closed)
                window.close();
            if (event.type == sf::Event::KeyPressed && event.key.code == sf::Keyboard::Space) {
                velocityY = JUMP_STRENGTH;
                jumpSound.play();
            }
        }

        velocityY += GRAVITY;
        bird.move(0, velocityY);

        if (pipes.empty() || pipes.back().x < WIDTH - 300) {
            float gapStart = 100 + rand() % (HEIGHT - PIPE_GAP - 200);
            Pipe pipe;
            pipe.x = WIDTH;
            pipe.upper.setSize(sf::Vector2f(PIPE_WIDTH, gapStart));
            pipe.upper.setPosition(pipe.x, 0);
            pipe.upper.setFillColor(sf::Color::Green);
            pipe.lower.setSize(sf::Vector2f(PIPE_WIDTH, HEIGHT - gapStart - PIPE_GAP));
            pipe.lower.setPosition(pipe.x, gapStart + PIPE_GAP);
            pipe.lower.setFillColor(sf::Color::Green);
            pipes.push_back(pipe);
        }

        for (auto &pipe : pipes) {
            pipe.x -= PIPE_SPEED;
            pipe.upper.setPosition(pipe.x, 0);
            pipe.lower.setPosition(pipe.x, pipe.upper.getSize().y + PIPE_GAP);
            
            if (pipe.x == bird.getPosition().x) {
                score++;
                pointSound.play();
            }
        }

        for (auto &pipe : pipes) {
            if (bird.getGlobalBounds().intersects(pipe.upper.getGlobalBounds()) ||
                bird.getGlobalBounds().intersects(pipe.lower.getGlobalBounds()) ||
                bird.getPosition().y >= HEIGHT) {
                hitSound.play();
                window.close();
            }
        }

        pipes.erase(remove_if(pipes.begin(), pipes.end(), [](Pipe &p) { return p.x + PIPE_WIDTH < 0; }), pipes.end());

        window.clear();
        window.draw(bird);
        for (auto &pipe : pipes) {
            window.draw(pipe.upper);
            window.draw(pipe.lower);
        }
        window.display();
    }
    return 0;
}
