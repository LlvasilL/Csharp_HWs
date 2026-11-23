# Задача 1
## Main

using System;
namespace homework;
class Program
{

    static void Main(string[] args)
    {

        Character player = new Character("Вася", 100, 10);

        player.ProcessTurn();

        player.ApplyEffect(new PoisonEffect());
        player.ApplyEffect(new BurnEffect());

        player.ProcessTurn();

        player.ApplyEffect(new PoisonEffect());
        player.ApplyEffect(new BurnEffect());
        player.ApplyEffect(new RegenerationEffect());
        player.ApplyEffect(new FreezeEffect());

        player.ProcessTurn();
        player.ProcessTurn();

    }

}
