#include <iostream>
#include <windows.h>
#include <thread>
#include <chrono>
#include <random>

// --- Globale Konfigurationsvariablen ---
// volatile, damit Änderungen sofort in allen Threads sichtbar sind
volatile bool g_isRunning = false;
int g_cps = 10;
UINT g_hotkey = VK_F6; // Standard-Hotkey: F6
bool g_isToggleMode = true;

// --- Verschleierung durch zufällige kleine Pausen ---
// Dies simuliert minimale menschliche Ungenauigkeit
void addRandomDelay() {
    static std::mt19937 rng(std::random_device{}());
    // Verteilung für eine sehr kleine Verzögerung, um das Timing nicht zu stark zu beeinflussen
    std::uniform_int_distribution<int> dist(0, 8); // 0-8 Millisekunden
    std::this_thread::sleep_for(std::chrono::milliseconds(dist(rng)));
}

// --- Funktion zum Senden eines Klicks ---
void sendClick() {
    // Simuliere einen echten Mausklick: Drücken und Loslassen
    mouse_event(MOUSEEVENTF_LEFTDOWN, 0, 0, 0, 0);
    // Eine sehr kurze, realistische Haltezeit für den Mausklick
    std::this_thread::sleep_for(std::chrono::milliseconds(10));
    mouse_event(MOUSEEVENTF_LEFTUP, 0, 0, 0, 0);
}

// --- Der Haupt-Thread für das Klicken ---
void clickerThread() {
    // Endlosschleife, die den Status prüft und klickt
    while (true) {
        if (g_isRunning) {
            sendClick();
            addRandomDelay(); // Füge eine kleine zufällige Verzögerung hinzu

            // Berechne die exakte Wartezeit, um die gewünschte CPS zu erreichen
            // 1000 ms / 10 cps = 100 ms pro Klick
            int delay = 1000 / g_cps;
            std::this_thread::sleep_for(std::chrono::milliseconds(delay));
        } else {
            // Wenn inaktiv, warte kurz, um CPU-Ressourcen zu schonen
            std::this_thread::sleep_for(std::chrono::milliseconds(50));
        }
    }
}

// --- Thread zur Überwachung der Tasteneingaben ---
void keyListenerThread() {
    while (true) {
        // Toggle-Modus (einmaliges Drücken zum Umschalten)
        if (g_isToggleMode) {
            // & 1 prüft, ob die Taste gerade gedrückt wurde (nicht ob sie gehalten wird)
            if (GetAsyncKeyState(g_hotkey) & 1) {
                g_isRunning = !g_isRunning; // Status umschalten
                // Visuelles Feedback im Konsolenfenster
                system("cls");
                std::cout << "Autoclicker ist " << (g_isRunning ? "AKTIV" : "INAKTIV") << std::endl;
                std::cout << "Hotkey: " << g_hotkey << " | CPS: " << g_cps << std::endl;
                std::cout << "Modus: " << (g_isToggleMode ? "Toggle" : "Hold") << std::endl;
                // Kurze Pause, um mehrfaches Umschalten bei einem langen Druck zu verhindern
                std::this_thread::sleep_for(std::chrono::milliseconds(200));
            }
        }
        // Hold-Modus (Klickt nur, während die Taste gehalten wird)
        else {
            if (GetAsyncKeyState(g_hotkey) < 0) { // < 0 bedeutet, die Taste ist aktuell gedrückt
                if (!g_isRunning) {
                    g_isRunning = true;
                    system("cls");
                    std::cout << "Autoclicker ist AKTIV (Halten zum Klicken)" << std::endl;
                    std::cout << "Hotkey: " << g_hotkey << " | CPS: " << g_cps << std::endl;
                    std::cout << "Modus: " << (g_isToggleMode ? "Toggle" : "Hold") << std::endl;
                }
            } else {
                if (g_isRunning) {
                    g_isRunning = false;
                    system("cls");
                    std::cout << "Autoclicker ist INAKTIV" << std::endl;
                    std::cout << "Hotkey: " << g_hotkey << " | CPS: " << g_cps << std::endl;
                    std::cout << "Modus: " << (g_isToggleMode ? "Toggle" : "Hold") << std::endl;
                }
            }
        }
        // Kurze Pause, um die CPU-Last dieses Threads gering zu halten
        std::this_thread::sleep_for(std::chrono::milliseconds(10));
    }
}

// --- Hauptfunktion zum Starten und Konfigurieren ---
int main() {
    // Konsolenfenster einrichten
    SetConsoleTitle(TEXT("Secure Autoclicker"));
    HANDLE hConsole = GetStdHandle(STD_OUTPUT_HANDLE);
    SetConsoleTextAttribute(hConsole, 10); // Grüne Textfarbe

    char choice;
    std::cout << "--- Autoclicker Konfiguration ---\n\n";

    // 1. Moduswahl
    std::cout << "Waehle den Modus:\n";
    std::cout << "1. An/Aus-Modus (Toggle)\n";
    std::cout << "2. Halten-Modus (Hold)\n";
    std::cout << "Wahl (1 oder 2): ";
    std::cin >> choice;
    g_isToggleMode = (choice == '1');

    // 2. CPS-Einstellung
    std::cout << "\nGib die Clicks pro Sekunde (CPS) ein (Standard 10): ";
    std::cin >> g_cps;
    if (g_cps <= 0) {
        g_cps = 10; // Auf Standard zurücksetzen, wenn ungültige Eingabe
    }

    // 3. Hotkey-Einstellung
    std::cout << "\nGib den virtuellen Keycode fuer den Hotkey ein.\n";
    std::cout << "Beispiele: 112=F1, 113=F2, 114=F3, 115=F4, 116=F5, 117=F6\n";
    std::cout << "Dein Keycode: ";
    int keyCode;
    std::cin >> keyCode;
    g_hotkey = static_cast<UINT>(keyCode);

    // Startbildschirm nach der Konfiguration
    system("cls");
    std::cout << "Autoclicker ist INAKTIV" << std::endl;
    std::cout << "Hotkey: " << g_hotkey << " | CPS: " << g_cps << std::endl;
    std::cout << "Modus: " << (g_isToggleMode ? "Toggle" : "Hold") << std::endl;
    std::cout << "\nDruecke die konfigurierte Taste zum Starten.\n";

    // Starte die Arbeits- und Listener-Threads
    std::thread clicker(clickerThread);
    std::thread listener(keyListenerThread);

    // Warte auf die Threads (diese laufen endlos)
    clicker.join();
    listener.join();

    return 0;
}
