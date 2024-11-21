#include <gb/gb.h>
#include <stdio.h>
#include <stdbool.h>

// Sprite-Daten
unsigned char player[] = {
    0x00, 0x3C, 0x42, 0x81, 0x81, 0xA5, 0x99, 0x42,
    0x42, 0x81, 0x99, 0xA5, 0x81, 0x42, 0x3C, 0x00
};

unsigned char crystal[] = {
    0x18, 0x3C, 0x7E, 0xFF, 0xFF, 0x7E, 0x3C, 0x18
};

unsigned char npc[] = {
    0x3C, 0x42, 0xA5, 0x81, 0x81, 0xA5, 0x99, 0x42
};

// Spielzustände
enum GAME_STATE { MENU, GAMEPLAY, DIALOGUE, COLLECTED_CRYSTAL, LEVEL_COMPLETE };
enum GAME_STATE current_state = MENU;

// Spielerposition
UINT8 player_x = 80;
UINT8 player_y = 72;

// Kristalle
struct Crystal {
    UINT8 x;
    UINT8 y;
    bool collected;
    const char* message;
};

struct Crystal crystals[3] = {
    {120, 72, false, "Freundschaft ist wichtig!\nTeile mit anderen."},
    {40, 120, false, "Ehrlichkeit zahlt sich aus.\nSei immer ehrlich!"},
    {80, 40, false, "Mut hilft dir, Herausforderungen zu meistern."}
};

// NPC-Daten
UINT8 npc_x = 60;
UINT8 npc_y = 72;
bool npc_encountered = false;

// Fortschritt
int current_level = 0;
int total_levels = 3;

// Menü anzeigen
void show_menu() {
    printf("Werte-Waechter\n");
    printf("Druecke START, um zu beginnen!\n");
}

// Dialog anzeigen
void show_dialogue(const char* message) {
    printf("%s\n", message);
}

// Spielerbewegung und Interaktionen
void gameplay() {
    // Bewegung
    if (joypad() & J_LEFT) player_x -= 2;
    if (joypad() & J_RIGHT) player_x += 2;
    if (joypad() & J_UP) player_y -= 2;
    if (joypad() & J_DOWN) player_y += 2;

    move_sprite(0, player_x, player_y);

    // Überprüfung auf Kristall
    struct Crystal* current_crystal = &crystals[current_level];
    if (!current_crystal->collected && player_x == current_crystal->x && player_y == current_crystal->y) {
        current_state = COLLECTED_CRYSTAL;
    }

    // Überprüfung auf NPC
    if (!npc_encountered && player_x == npc_x && player_y == npc_y) {
        npc_encountered = true;
        current_state = DIALOGUE;
    }
}

// Kristall einsammeln
void collect_crystal() {
    struct Crystal* current_crystal = &crystals[current_level];
    current_crystal->collected = true;
    printf("Du hast einen Kristall gefunden!\n");
    printf("%s\n", current_crystal->message);

    // Fortschritt prüfen
    current_level++;
    if (current_level >= total_levels) {
        current_state = LEVEL_COMPLETE;
    } else {
        current_state = GAMEPLAY;
    }
}

// NPC-Dialog
void npc_dialogue() {
    printf("NPC: Nur wer Respekt zeigt, kommt weiter.\n");
    printf("Antwort A: Ich zeige Respekt. (Richtig)\n");
    printf("Antwort B: Geh aus dem Weg! (Falsch)\n");

    if (joypad() & J_A) {
        printf("NPC: Gute Antwort. Du kannst passieren.\n");
        current_state = GAMEPLAY;
    }
    if (joypad() & J_B) {
        printf("NPC: Falsche Einstellung. Versuche es erneut.\n");
        current_state = GAMEPLAY;
    }
}

// Level abschließen
void complete_level() {
    printf("Glückwunsch! Du hast alle Kristalle gefunden.\n");
    printf("Danke, dass du die Werte bewahrst.\n");
    printf("Drücke START, um neu zu starten.\n");

    if (joypad() & J_START) {
        current_level = 0;
        for (int i = 0; i < total_levels; i++) {
            crystals[i].collected = false;
        }
        current_state = MENU;
    }
}

// Hauptfunktion
void main() {
    // Initialisierung
    SPRITES_8x8;
    set_sprite_data(0, 1, player);
    set_sprite_data(1, 1, crystal);
    set_sprite_data(2, 1, npc);
    set_sprite_tile(0, 0); // Spieler
    set_sprite_tile(1, 1); // Kristall
    set_sprite_tile(2, 2); // NPC

    move_sprite(0, player_x, player_y);
    SHOW_SPRITES;

    while (1) {
        switch (current_state) {
            case MENU:
                show_menu();
                if (joypad() & J_START) {
                    current_state = GAMEPLAY;
                    printf("\n");
                }
                break;

            case GAMEPLAY:
                gameplay();
                struct Crystal* current_crystal = &crystals[current_level];
                if (!current_crystal->collected) move_sprite(1, current_crystal->x, current_crystal->y);
                else hide_sprite(1);
                break;

            case COLLECTED_CRYSTAL:
                collect_crystal();
                break;

            case DIALOGUE:
                npc_dialogue();
                break;

            case LEVEL_COMPLETE:
                complete_level();
                break;
        }
        delay(100);
    }
}

