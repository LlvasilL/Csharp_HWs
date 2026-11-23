# Задача 1
## Main
```
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
```
## interface IStatusEffect
```
using System;
namespace homework
{
	public interface IStatusEffect
	{
		string Name { get; }
		int Duration { get; set; }

		bool CanStack { get; }
		bool JustApplied { get; set; }

		bool ReduceDuration();
		void ApplyEffect(Character target);
    }
}
```
## class Character
```
using System;
using System.Collections.Generic;
namespace homework
{
	public class Character
	{
		private string name;
		private int hp;
        private int armor;

        public int MaxHP { get; private set; }
		public bool IsFrozen { get; set; } = false;

		private List<IStatusEffect> ActiveEffects = new List<IStatusEffect>();

		public string Name { get { return name; } }
		public int HP { get { return hp; } set { hp = Math.Max(0, Math.Min(MaxHP, value)); } }
		public int Armor { get { return armor; } }

        public Character(string inputName, int inputMaxHP, int inputArmor)
		{
			name = inputName;
			MaxHP = inputMaxHP;
			armor = inputArmor;
			hp = inputMaxHP;
		}

		public void TakeDamage(int amount)
		{
			HP -= amount;
		}

		public void Heal(int amount)
		{
			HP += amount;
		}

		public void ProcessTurn()
		{
			

			if (IsFrozen)
			{
				Console.WriteLine($"Персонаж {Name} заморожен и пропускает ход");
			}

			for(int i = ActiveEffects.Count - 1; i >= 0; i--)
			{
				IStatusEffect effect = ActiveEffects[i];

				if (effect.JustApplied)
				{
					effect.JustApplied = false;
					continue;
				}

				effect.ApplyEffect(this);

				if (effect.ReduceDuration())
				{
					if (effect is FreezeEffect) IsFrozen = false;
					ActiveEffects.RemoveAt(i);
					Console.WriteLine($"Эффект [{effect.Name}] истёк.");
				}
			}

			Console.WriteLine($"——————————————— Состояние персонажа: ————————————————");
			Console.WriteLine($"Здоровье: {HP}/{MaxHP}, броня: {Armor}");
			if(ActiveEffects.Count > 0)
			{
				Console.Write($"Активные эффекты: ");
				foreach(var e in ActiveEffects)
				{
					Console.Write($"Эффект: [{e.Name}], оставшаяся длительность: [{e.Duration}]; ");
				}
				Console.WriteLine();
			}
			else Console.WriteLine("Активных эффектов нет");
			Console.WriteLine();
		}

		public void ApplyEffect(IStatusEffect effect)
		{
            
            bool FindSameEffect = false;

			foreach(IStatusEffect activeEffect in ActiveEffects)
			{
				if(activeEffect.Name == effect.Name)
				{
					if (activeEffect.CanStack)
					{
                        activeEffect.Duration += effect.Duration;
						Console.WriteLine($"Длительность эффекта [{effect.Name}] повышена");
                    }
					else
					{
						activeEffect.Duration = effect.Duration;
                        Console.WriteLine($"Эффект [{effect.Name}] обновлён");
                    }
                    FindSameEffect = true;
                    break;
                }	
			}
			if(FindSameEffect == false)
			{
				effect.JustApplied = true;
				ActiveEffects.Add(effect);
				Console.WriteLine($"Наложен эффект [{effect.Name}] на [{effect.Duration}] ходов");
			}
			
		}
	}
}
```
## Effects
```
using System;
namespace homework
{
    public class PoisonEffect : IStatusEffect
    {
        private string name = "Отравление";
        private int duration = 3;
        private bool canStack = true;
        private int damage = 10;



        public string Name { get => name; }

        public int Duration { get => duration; set => duration = value; }

        public bool CanStack { get => canStack; }

        public bool JustApplied { get; set; }

        public void ApplyEffect(Character target)
        {
            target.TakeDamage(damage);
            Console.WriteLine($"Игрок {target.Name} отравлен. Получено {damage} урона.");

        }

        public bool ReduceDuration()
        {
            Duration -= 1;

            if (Duration <= 0)
            {
                return true;
            }
            else return false;

        }
    }

    public class BurnEffect : IStatusEffect
    {
        private string name = "Горение";
        private int duration = 2;
        private bool canStack = false;
        private int damage = 15;



        public string Name { get => name; }

        public int Duration { get => duration; set => duration = value; }

        public bool CanStack { get => canStack; }

        public bool JustApplied { get; set; }

        public void ApplyEffect(Character target)
        {
            int actualDamage = Math.Max(0, damage - target.Armor);
            target.TakeDamage(actualDamage);
            Console.WriteLine($"Игрок {target.Name} горит. Получено {actualDamage} урона.");
        }

        public bool ReduceDuration()
        {
            Duration -= 1;

            if (Duration <= 0)
            {
                return true;
            }
            else return false;

        }
    }

    public class RegenerationEffect : IStatusEffect
    {
        private string name = "Регенерация";
        private int duration = 4;
        private bool canStack = false;
        private int heal = 5;

        public string Name { get => name; }

        public int Duration { get => duration; set => duration = value; }

        public bool CanStack { get => canStack; }

        public bool JustApplied { get; set; }

        public void ApplyEffect(Character target)
        {
            int firstHp = target.HP;
            target.HP = Math.Min(target.MaxHP, target.HP + heal);
            Console.WriteLine($"Игрок {target.Name} хилиться. Восстановлено {target.HP - firstHp} здоровья.");
        }

        public bool ReduceDuration()
        {
            Duration -= 1;

            if (Duration <= 0)
            {
                return true;
            }
            else return false;

        }
    }

    public class FreezeEffect : IStatusEffect
    {
        private string name = "Заморозка";
        private int duration = 1;
        private bool canStack = false;

        public string Name { get => name; }

        public int Duration { get => duration; set => duration = value; }

        public bool CanStack { get => canStack; }

        public bool JustApplied { get; set; }

        public void ApplyEffect(Character target)
        {
            target.IsFrozen = true;
        }

        public bool ReduceDuration()
        {
            Duration -= 1;

            if (Duration <= 0)
            {
                return true;
            }
            else return false;

        }
    }
}
```
# Задача 2
## Main
```
using System;
namespace homework;
class Program
{

    static void Main(string[] args)
    {
        Player player = new Player("Вася", 100);
        Enemy enemy = new Enemy("Враг", 80);
        Ally ally = new Ally("Компаньон", 50);

        Chest chest = new Chest();
        Trap trap = new Trap();
        Campfire campfire = new Campfire();

        List<IEntity> entities = new List<IEntity> { player, enemy, ally };
        List<IEnvironmentObject> objects = new List<IEnvironmentObject> { chest, trap, campfire };

        foreach(var entity in entities)
        {
            foreach(var obj in objects)
            {
                entity.InteractWith(obj);
            }
        }
    }

}
```
## interface IEntity
```
using System;
namespace homework
{
	public interface IEntity
	{
		string Name { get; }
		int HP { get; set; }
		void InteractWith(IEnvironmentObject obj);
    }

	public interface IEnvironmentObject
	{
		void Accept(IEntity entity);
	}

	public interface IEntityVisitor
	{
        void VisitChest(Chest chest);
        void VisitTrap(Trap trap);
        void VisitCampfire(Campfire campfire);
    }
}
```
## Классы
```
using System;
using System.Collections.Generic;
namespace homework
{
	public class Player: IEntity, IEntityVisitor
    {
		private string name;
		private int hp;


		public string Name { get { return name; } }
		public int HP { get { return hp; } set { hp = Math.Max(0, value); } }

        public Player(string inputName, int inputHP)
		{
			name = inputName;
			hp = inputHP;
		}

        public void InteractWith(IEnvironmentObject obj)
		{
            obj.Accept(this);
		}
        public void VisitChest(Chest chest)
        {
            Console.WriteLine($"Игрок {Name} открыл сундук и забрал золото");
        }
        public void VisitTrap(Trap trap)
        {
            Random random = new Random();
            bool defuseTrap = random.Next(2) == 0;
            if (defuseTrap)
            {
                Console.WriteLine($"Игрок {Name} обезвредил ловушку");
                trap.IsDisarmed = true;
            }
            else
            {
                Console.WriteLine($"Игрок {Name} попался в ловушку. и получил 30 урона");
                HP -= 30;
            }
            
        }
        public void VisitCampfire(Campfire campfire)
        {
            Console.WriteLine($"Игрок {Name} зажёг костёр");
            campfire.IsFire = true;

        }

    }

    public class Enemy : IEntity, IEntityVisitor
    {
        private string name;
        private int hp;


        public string Name { get { return name; } }
        public int HP { get { return hp; } set { hp = Math.Max(0, value); } }

        public Enemy(string inputName, int inputHP)
        {
            name = inputName;
            hp = inputHP;
        }

        public void InteractWith(IEnvironmentObject obj)
        {
            obj.Accept(this);
        }
        public void VisitChest(Chest chest)
        {
            Console.WriteLine($"Зомби {Name} не может открыть сундук");
        }
        public void VisitTrap(Trap trap)
        {
            Console.WriteLine($"Зомби {Name} попался в ловушку. и получил 30 урона");
            HP -= 30;
        }
        public void VisitCampfire(Campfire campfire)
        {
            Console.WriteLine($"Зомби {Name} потушил костёр");
            campfire.IsFire = false;
        }
    }

    public class Ally : IEntity, IEntityVisitor
    {
        private string name;
        private int hp;


        public string Name { get { return name; } }
        public int HP { get { return hp; } set { hp = Math.Max(0, value); } }

        public Ally(string inputName, int inputHP)
        {
            name = inputName;
            hp = inputHP;
        }

        public void InteractWith(IEnvironmentObject obj)
        {
            obj.Accept(this);
        }
        public void VisitChest(Chest chest)
        {
            Console.WriteLine($"Союзник {Name} не может открыть сундук");
        }
        public void VisitTrap(Trap trap)
        {
            Console.WriteLine($"Союзник {Name} попался в ловушку и получил 30 урона");
            HP -= 30;
        }
        public void VisitCampfire(Campfire campfire)
        {
            Console.WriteLine($"Союзник {Name} поджёгся");
            HP -= 10;
        }
    }

    public class Chest: IEnvironmentObject
    {
        public void Accept(IEntity entity)
        {
            if (entity is IEntityVisitor visitor)
            {
                visitor.VisitChest(this);
            }
        }
    }

    public class Trap: IEnvironmentObject
    {
        public bool IsDisarmed { get; set; } = false;

        public void Accept(IEntity entity)
        {
            if (entity is IEntityVisitor visitor)
            {
                visitor.VisitTrap(this);
            }
        }
    }

    public class Campfire : IEnvironmentObject
    {
        public bool IsFire { get; set; } = false;

        public void Accept(IEntity entity)
        {
            if (entity is IEntityVisitor visitor)
            {
                visitor.VisitCampfire(this);
            }
        }
    }
}
```


