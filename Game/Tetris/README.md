#include <SFML/Graphics.hpp>
#include <time.h>

using namespace sf;

const int M = 20;// висота ігрового поля
const int N = 10;// ширина ігрового поля

int field[M][N] = { 0 };// ігрове поле
int w = 34;

struct Point
{
    int x, y;
}a[4], b[4];

// Масив фігурок-тетраміно
int figures[7][4] = {
    1,3,5,7, // I
    2,4,5,7, // S
    3,5,4,6, // Z
    3,5,4,7, // T
    2,3,5,7, // L
    3,5,7,6, // J
    2,3,4,5, // O
};

// Перевірка на вихід за межі ігрового поля
bool check() {
    for (int i = 0; i < 4; i++)
        if (a[i].x < 0 || a[i].x >= N || a[i].y >= M)
            return 0;
        else if (field[a[i].y][a[i].x])
            return 0;

    return 1;
}

int main()
{
    srand(time(0));

    RenderWindow window(VideoMode(N * w, M * w), "Tetris!");

    // Створення та завантаження текстури
    Texture t;
    t.loadFromFile("C:/Users/kreho/OneDrive/Рабочий стол/tiles.jpg");
    // Створення спрайту
    Sprite tiles(t);

    // Змінні для горизонтального переміщення та обертання
    int dx = 0;// Змінна для горизонтального переміщення тетраміно
    int colorNum = 1;
    bool rotate = false;// змінна для обертання тетраміно
    // Змінні для таймера та затримки
    float timer = 0, delay = 0.3;
    // Годинник (таймер)
    Clock clock;
    bool ad = true;

    // Головний цикл програми: виконується, поки відкрито вікно
    while (window.isOpen())
    {
        // Отримуємо час, що минув з початку відліку, і конвертуємо його за секунди
        float time = clock.getElapsedTime().asSeconds();
        clock.restart();
        timer += time;

        // Обробляємо події у циклі
        Event event;
        while (window.pollEvent(event))
        {
            // Користувач натиснув на хрестик і хоче закрити вікно?
            if (event.type == Event::Closed)
                // тоді закриваємо його
                window.close();

            // Чи була натиснута кнопка на клавіатурі?
            if (event.type == Event::KeyPressed)
                // Ця кнопка – стрілка вгору?
                if (event.key.code == Keyboard::Up)
                    rotate = true;
            // Або стрілка праворуч?
                else if (event.key.code == Keyboard::Right)
                    dx = 1;
            // Чи може стрілка вліво?
                else if (event.key.code == Keyboard::Left)
                    dx = -1;
        }


        if (Keyboard::isKeyPressed(Keyboard::Down))
            delay = 0.05;

        // Горизонтальне переміщення
        for (int i = 0; i < 4; i++) {
            b[i] = a[i];
            a[i].x += dx;
        }

        // Якщо вийшли за межі поля після переміщення, то повертаємо старі координати 
        if (!check()) {
            for (int i = 0; i < 4; i++)
                a[i] = b[i];
        }

        // обертання
        if (rotate) {
            Point p = a[1];// вказуємо центр обертання
            for (int i = 0; i < 4; i++) {
                int x = a[i].y - p.y;// y - y0
                int y = a[i].x - p.x;// x - x0

                a[i].x = p.x - x;
                a[i].y = p.y + y;
            }

            // Якщо вийшли межі поля після повороту, то повертаємо старі координати
            if (!check()) {
                for (int i = 0; i < 4; i++)
                    a[i] = b[i];
            }
        }

        // Рух тетраміно вниз (<тік> таймера)
        if (timer > delay) {
            // Горизонтальне переміщення
            for (int i = 0; i < 4; i++) {
                b[i] = a[i];
                a[i].y += 1;
            }

            // Якщо вийшли за межі поля після переміщення, то повертаємо старі координати
            if (!check()) {
                for (int i = 0; i < 4; i++)
                    field[b[i].y][b[i].x] = colorNum;
                colorNum = 1 + rand() % 7;
                int n = rand() % 7;// задаємо тип тетраміно
                // Перша поява тетраміно на полі?
                for (int i = 0; i < 4; i++) {
                    a[i].x = figures[n][i] % 2;
                    a[i].y = figures[n][i] / 2;
                }
            }

            timer = 0;
        }

        // Перша поява тетраміно на полі?
        if (ad) {
            int n = rand() % 7;// вказуємо тип тетраміно
            // Перша поява тетраміно на полі?
            if (a[0].x == 0)
                for (int i = 0; i < 4; i++) {
                    a[i].x = figures[n][i] % 2;
                    a[i].y = figures[n][i] / 2;
                }
            ad = false;
        }

        int k = M - 1;
        for (int i = M - 1; i > 0; i--) {
            int count = 0;
            for (int j = 0; j < N; j++) {
                if (field[i][j])
                    count++;
                field[k][j] = field[i][j];
            }
            if (count < N)
                k--;
        }

        dx = 0; // горизонтальне переміщення
        rotate = false;// обертання
        delay = 0.3;// колір

        // Відображення вікна
        window.clear(Color::White);

        for (int i = 0; i < M; i++)
            for (int j = 0; j < N; j++) {
                if (field[i][j] == 0)
                    continue;
                tiles.setTextureRect(IntRect(field[i][j] * w, 0, w, w));
                tiles.setPosition(j * w, i * w);
                window.draw(tiles);
            }

        for (int i = 0; i < 4; i++) {
            tiles.setTextureRect(IntRect(colorNum * w, 0, w, w));
            // Встановлюємо позицію кожного шматочка тетраміно
            tiles.setPosition(a[i].x * w, a[i].y * w);
            // Відображення спрайту
            window.draw(tiles);
        }

        // Відображення вікна
        window.display();
    }

    return 0;
}

