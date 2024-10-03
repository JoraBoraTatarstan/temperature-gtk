#include <gtk/gtk.h>
#include <stdio.h>
#include <stdlib.h>

static void get_temperature(GtkWidget *label) {
    FILE *temperatureFile;
    double temperature;
    
    // Открываем файл с температурой
    temperatureFile = fopen("/sys/class/thermal/thermal_zone0/temp", "r");
    
    if (temperatureFile == NULL) {
        gtk_label_set_text(GTK_LABEL(label), "Не удалось открыть файл с температурой.");
        return;
    }
    
    // Чтение температуры
    fscanf(temperatureFile, "%lf", &temperature);
    fclose(temperatureFile);
    
    // Преобразование температуры в градусы Цельсия
    temperature /= 1000;

    // Форматируем строку для вывода
    char temp_str[64];
    snprintf(temp_str, sizeof(temp_str), "Температура процессора: %.2f°C", temperature);

    // Устанавливаем текст метки
    gtk_label_set_text(GTK_LABEL(label), temp_str);
}

static void on_button_clicked(GtkWidget *widget, gpointer label) {
    get_temperature(GTK_WIDGET(label));
}

int main(int argc, char *argv[]) {
    // Инициализация GTK
    gtk_init(&argc, &argv);

    // Создание главного окна
    GtkWidget *window = gtk_window_new(GTK_WINDOW_TOPLEVEL);
    gtk_window_set_title(GTK_WINDOW(window), "Отслеживание температуры");
    gtk_window_set_default_size(GTK_WINDOW(window), 300, 200);

    // Создание вертикального контейнера
    GtkWidget *vbox = gtk_box_new(GTK_ORIENTATION_VERTICAL, 5);
    gtk_container_add(GTK_CONTAINER(window), vbox);

    // Создание метки
    GtkWidget *label = gtk_label_new("Нажмите кнопку для получения температуры");
    gtk_box_pack_start(GTK_BOX(vbox), label, TRUE, TRUE, 0);

    // Создание кнопки
    GtkWidget *button = gtk_button_new_with_label("Получить температуру");
    gtk_box_pack_start(GTK_BOX(vbox), button, TRUE, TRUE, 0);

    // Привязка события нажатия кнопки
    g_signal_connect(button, "clicked", G_CALLBACK(on_button_clicked), label);

    // Привязка события закрытия окна
    g_signal_connect(window, "destroy", G_CALLBACK(gtk_main_quit), NULL);

    // Отображение всех элементов
    gtk_widget_show_all(window);

    // Запуск основного цикла GTK
    gtk_main();

    return 0;
}
