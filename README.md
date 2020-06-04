-- zombieapocalype.scar
-- Created all by Cylarne @ 2012

import("ScarUtil.scar");

--[[TODO:]]
-- Zombie Setup = CHECK
-- Zombie create = CHECK
-- Check Zombies = CHECK
-- Check Bitten = CHECK
-- Check Lose = CHECK
-- Zombietactics = CHECK
-- Reinforcements = CHECK
-- Large waves = CHECK

function ZombieApocalypse()

	-- How many survivor? Yes, this is the checker for the lose condition.
	g_survivor_count = 2;
	g_survivor_count_total = 0;
	
	-- Zombie Squad Groups, this is used for spawnng more and more zombies.
	g_zombiecount = 0;
	
	-- Integers for checking zombie players.
	Player1_IsZombie = false;
	Player2_IsZombie = false;
	Player3_IsZombie = false;
	Player4_IsZombie = false;
	Player5_IsZombie = false;
	Player6_IsZombie = false;
	Player7_IsZombie = false;
	Player8_IsZombie = false;
	
	ZombieApocalypse_Setup(); -- Setup function.
	Rule_AddInterval(ZombieApocalypse_CheckZombies, 0.5) -- Function that checks for dead zombies, for the zombies that are dead, they respawn in many amounts over time.
	Rule_AddInterval(ZombieApocalypse_CheckSquadsDead, 0.5) -- Lose rule, if no squads are left, the player loses.
	Rule_AddInterval(ZombieApocalypse_ZombieTactics, 30) -- Every 60 seconds, zombies randomly goes to starting position to starting position or attacks a player squad. Or does nothing.
	Rule_AddInterval(ZombieApocalypse_Reinforcements, 9999) -- Player gets spare reinforcements.
	Rule_AddInterval(ZombieApocalypse_MassiveWaves, g_massivewaves_timer) -- Massive armies of zombies marches toward the survivors every 10 minutes+.
	
	-- Create the ALPHA Zombie.
	Util_CreateSquadsAtPosition(g_DefaultZombies, "ork_squad_warboss", "ork_squad_slugga", Player_GetStartPosition(g_DefaultZombies), 1);
	
	Rule_AddInterval(ZombieApocalypse_AlphaZombie_Reinforcements, 90) -- Alpha Zombie spawns reinforcements of zombies over time.
	Rule_AddInterval(ZombieApocalypse_AlphaZombie_Tactic, 40) -- Alpha Zombie moves to places.
	
	
	obj_table_zombieapocalypse_alphazombie = { title_id = 16600000, short_desc_id = 16600001, help_tip_id = 16600002 }
	
	Objective_Add( obj_table_zombieapocalypse_alphazombie, true )

	g_DefaultSurvivor_kills = Stats_PlayerUnitsKilled(Player_GetID(g_DefaultSurvivor))
	UI_ShowRatio("Kills", g_DefaultSurvivor, 16600021, g_DefaultSurvivor_kills, i_zombiekillcount )
	Rule_AddInterval(ZombieApocalypse_Survival_KillCount, 0.1)
	
end

function ZombieApocalypse_Survival_KillCount()
	
	if (g_DefaultSurvivor_kills < i_zombiekillcount) then
		g_DefaultSurvivor_kills = Stats_PlayerUnitsKilled(Player_GetID(g_DefaultSurvivor))
		UI_ShowRatioUpdate("Kills", 16600021, g_DefaultSurvivor_kills, i_zombiekillcount )
	elseif (g_DefaultSurvivor_kills >= i_zombiekillcount) then
		Player_Kill(g_DefaultZombies)
		World_SetTeamWin(g_DefaultSurvivor, "annihilate")
		World_SetGameOver()
	end
	
end

function ZombieApocalypse_AlphaZombie_Reinforcements()
	
	World_FXEventSquad("data:Art/Events/Chaos/Blood_Pulse", "ork_squad_warboss")
	Util_CreateSquadsAtPosition(g_DefaultZombies, "sg_zombie1", "ork_squad_slugga", SGroup_GetPosition("ork_squad_warboss"), i_alphazombiereinforcement);
	Util_CreateSquadsAtPosition(g_DefaultZombies, "sg_zombie1", "ork_squad_burnaz", SGroup_GetPosition("ork_squad_warboss"), i_alphazombiereinforcement_2);
	
end

function ZombieApocalypse_AlphaZombie_Tactic()

	g_alphazombietactic = World_GetRand(1,1000);
	
	if (SGroup_Exists("ork_squad_warboss")) then
		if (g_alphazombietactic >= 850) then
			Cmd_AttackMovePos("ork_squad_warboss", Player_GetStartPosition(g_DefaultZombies));
		elseif (g_alphazombietactic <= 150) then
			Cmd_StopSquads("ork_squad_warboss")
		else
		end
		g_alphazombietactic = World_GetRand(1,1000);
	end
end

function ZombieApocalypse_Setup()
	
	
	-- Destroy all starting hqs and starting positions.
	local count = World_GetPlayerCount();
	for j = 0, count-1 do
		local Player = World_GetPlayerAt(j);
		
		-- Destroy starting hq and builder unit. Idea thanks to Jaguar Lord.
		Player_GetAllEntitiesNearPos(Player, "eg_destroy_player_entities", Player_GetStartPosition(Player), 15)
		Player_GetAllSquadsNearPos(Player, "sg_destroy_player_squad", Player_GetStartPosition(Player), 15)
		EGroup_DeSpawn("eg_destroy_player_entities")
		SGroup_DeSpawn("sg_destroy_player_squad")
		
		EGroup_DestroyAllEntities("eg_destroy_player_entities", true)
		SGroup_DestroyAllSquads("sg_destroy_player_squad", true)
		
		-- Is Zombie? Then set the zombies at base, and set the instance variable "Zombie?" to true.
		if (Player_IsAlive(Player) and Cpu_IsCpuPlayer(Player)) then -- Basic function for checking zombie players.
			
			Cpu_Enable(Player, false); -- Disable Cpu
			
			g_DefaultZombies = Player;
			-- Difficulty settings.
			if (Cpu_GetDifficulty(g_DefaultZombies) == AD_Easy) then
				i_zombiecountdifficulty = 1; -- Every zombie squad group killed, adds this amount per term.
				g_massivewaves_timer = 1200; -- How many seconds it takes to spawn a massive wave of zombies.
				g_zombiecount_starting = 4; -- How many zombies SGroups spawns on game start.
				g_zombiehealth_modifier = 0.75; -- Zombie health modifier number.
				i_alphazombiereinforcement = 4; -- ALPHA zombie rules, zombie amount spawned every x seconds.
				i_alphazombiereinforcement_2 = 2; -- ALPHA zombie rules, fast zombie amount spawned every x seconds.
				i_zombiekillcount = 350; -- Survival rules, the required kills needed to pass the mission.
			elseif (Cpu_GetDifficulty(g_DefaultZombies) == AD_Standard) then
				i_zombiecountdifficulty = 2;
				g_massivewaves_timer = 800;
				g_zombiecount_starting = 5;
				g_zombiehealth_modifier = 1.0;
				i_alphazombiereinforcement = 6;
				i_alphazombiereinforcement_2 = 2;
				i_zombiekillcount = 700;
			elseif (Cpu_GetDifficulty(g_DefaultZombies) == AD_Hard) then
				i_zombiecountdifficulty = 2;
				g_massivewaves_timer = 500;
				g_zombiecount_starting = 6;
				g_zombiehealth_modifier = 1.0;
				i_alphazombiereinforcement = 8;
				i_alphazombiereinforcement_2 = 4;
				i_zombiekillcount = 850;
			elseif (Cpu_GetDifficulty(g_DefaultZombies) == AD_Advanced) then
				i_zombiecountdifficulty = 3;
				g_massivewaves_timer = 450;
				g_zombiecount_starting = 8;
				g_zombiehealth_modifier = 1.2;
				i_alphazombiereinforcement = 8;
				i_alphazombiereinforcement_2 = 6;
				i_zombiekillcount = 2000;
			elseif (Cpu_GetDifficulty(g_DefaultZombies) == AD_Insane) then
				i_zombiecountdifficulty = 3;
				g_massivewaves_timer = 300;
				g_zombiecount_starting = 10;
				g_zombiehealth_modifier = 1.3;
				i_alphazombiereinforcement = 8;
				i_alphazombiereinforcement_2 = 6;
				i_zombiekillcount = 9999;
			end
			
			-- Setup the zombies! Bwa ha ha!
			for i = 1, g_zombiecount_starting do
				Util_CreateSquadsAtPositionRandom(Player, "sg_zombie"..i, "ork_squad_slugga", Player_GetStartPosition(Player), 1);
				g_zombiecount = g_zombiecount + 1;
			end
			if (j == 0) then Player1_IsZombie = true; g_Player1 = World_GetPlayerAt(0);
			elseif (j == 1) then Player2_IsZombie = true; g_Player2 = World_GetPlayerAt(1);
			elseif (j == 2) then Player3_IsZombie = true; g_Player3 = World_GetPlayerAt(2);
			elseif (j == 3) then Player4_IsZombie = true; g_Player4 = World_GetPlayerAt(3);
			elseif (j == 4) then Player5_IsZombie = true; g_Player5 = World_GetPlayerAt(4);
			elseif (j == 5) then Player6_IsZombie = true; g_Player6 = World_GetPlayerAt(5);
			elseif (j == 6) then Player7_IsZombie = true; g_Player7 = World_GetPlayerAt(6);
			elseif (j == 7) then Player8_IsZombie = true; g_Player8 = World_GetPlayerAt(7);
			end
			
			local oHealthModifier	= Modifier_Create(MAT_Player, "health_maximum_player_modifier", MUT_Multiplication, false, g_zombiehealth_modifier, "")
			Modifier_ApplyToPlayer(oHealthModifier, Player)
			
			-- Set all zombies as team mates.
			Player_SetTeam(Player, 1);
		-- Is human? Then set the Tactical Marines and civilians at start, set resource rates and resources to 0.
		elseif (Player_IsAlive(Player)) then
			Player_SetResource(Player, RT_Power, 9999);
			Player_SetResource(Player, RT_Requisition, 9999);
			local reqmodifier = Modifier_Create(MAT_Player, "income_requisition_player_modifier", MUT_Multiplication, false, 2, "")  
			local powermodifier = Modifier_Create(MAT_Player, "income_power_player_modifier", MUT_Multiplication, false, 1, "")  
			
			if (Player_GetRaceName(Player) == "chaos_marine_race") then
				blueprint_commander = "chaos_squad_lord";
				blueprint_squad = "chaos_squad_chaos_terminator";
				blueprint_builder_squad = "chaos_squad_slave";
				Player_GrantResearch (Player,"chaos_wargear01");
				Player_GrantResearch (Player,"chaos_wargear02");
				Player_GrantResearch (Player,"chaos_wargear03");
				Player_GrantResearch (Player,"chaos_wargear04");
				Player_GrantResearch (Player,"chaos_wargear05");
				Player_GrantResearch (Player,"chaos_wargear06");
				Player_GrantResearch (Player,"chaos_wargear07");
				Player_GrantResearch (Player,"chaos_wargear08");
				Player_GrantResearch (Player,"chaos_wargear09");
				Player_GrantResearch (Player,"mark_chaos");
				Player_GrantResearch (Player,"mark_chaos_khorne");
				Player_GrantResearch (Player,"mark_chaos_nurgle");
				Player_GrantResearch (Player,"mark_chaos_slaanesh");
				Player_GrantResearch (Player,"mark_chaos_tzeentch");
				Player_GrantResearch (Player,"chaos_commander_health_research_1");
				Player_GrantResearch (Player,"chaos_commander_health_research_2");
				Player_GrantResearch (Player,"chaos_health_upgrade_research");
				Player_GrantResearch (Player,"chaos_health_upgrade_research_2");
				Player_GrantResearch (Player,"chaos_lord_research_1");
				Player_GrantResearch (Player,"chaos_lord_research_2");
				Player_GrantResearch (Player,"chaos_frag_grenade_research");
				Player_GrantResearch (Player,"chaos_plasma_pistol_research");
				Player_GrantResearch (Player,"chaos_champion_melee_research_1");
				Player_GrantResearch (Player,"chaos_champion_melee_research_2");
				Player_GrantResearch (Player,"chaos_max_weapons_research");
				Player_GrantResearch (Player,"chaos_accuracy_upgrade_research");
				Player_GrantResearch (Player,"chaos_accuracy_upgrade_research_2");
		
			-- Dark Eldar
			elseif (Player_GetRaceName(Player) == "dark_eldar_race") then
				blueprint_commander = "dark_eldar_squad_archon_survival";
				blueprint_squad = "dark_eldar_squad_warrior";
				blueprint_builder_squad = "dark_eldar_squad_slave";
				Player_GrantResearch (Player,"dark_eldar_wargear01");
				Player_GrantResearch (Player,"dark_eldar_wargear02");
				Player_GrantResearch (Player,"dark_eldar_wargear03");
				Player_GrantResearch (Player,"dark_eldar_wargear04");
				Player_GrantResearch (Player,"dark_eldar_wargear05");
				Player_GrantResearch (Player,"dark_eldar_wargear06");
				Player_GrantResearch (Player,"dark_eldar_wargear07");
				Player_GrantResearch (Player,"dark_eldar_wargear08");
				Player_GrantResearch (Player,"dark_eldar_wargear09");
				Player_GrantResearch (Player,"dark_eldar_wargear10");
				Player_GrantResearch (Player,"dark_eldar_crucible_of_malediction_research");
				Player_GrantResearch (Player,"dark_eldar_poison_blades_research");
				Player_GrantResearch (Player,"dark_eldar_range_increase_research");
				Player_GrantResearch (Player,"dark_eldar_stinger_research");
				Player_GrantResearch (Player,"dark_eldar_upgrade_agonizer");
				Player_GrantResearch (Player,"dark_eldar_upgrade_commander_health");
				Player_GrantResearch (Player,"dark_eldar_upgrade_commander_health_2");
				Player_GrantResearch (Player,"dark_eldar_upgrade_infantry_health");
				Player_GrantResearch (Player,"dark_eldar_upgrade_destructor");
				Player_GrantResearch (Player,"dark_eldar_upgrade_retinue_size_1");
				Player_GrantResearch (Player,"dark_eldar_upgrade_retinue_size_2");
				Player_GrantResearch (Player,"dark_eldar_upgrade_retinue_size_3");
				Player_GrantResearch (Player,"dark_eldar_upgrade_soulseeker_ammunition");
				
			-- Eldar	
			elseif (Player_GetRaceName(Player) == "eldar_race") then
				blueprint_commander = "eldar_squad_farseer";
				blueprint_squad = "eldar_squad_dark_reapers";
				blueprint_builder_squad = "eldar_squad_bonesinger";
				Player_GrantResearch (Player,"eldar_bonesinger_ability_research_1");
				Player_GrantResearch (Player,"eldar_farseer_ability_research");
				Player_GrantResearch (Player,"eldar_farseer_ability_research_2");
				Player_GrantResearch (Player,"eldar_farseer_ability_research_3");
				Player_GrantResearch (Player,"eldar_fleet_of_foot_research");
				Player_GrantResearch (Player,"eldar_haywire_bomb_research");
				Player_GrantResearch (Player,"eldar_research_farseerhealth_1");
				Player_GrantResearch (Player,"eldar_research_farseerhealth_2");
				Player_GrantResearch (Player,"eldar_research_infantryaccuracy_1");
				Player_GrantResearch (Player,"eldar_research_infantryaccuracy_2");
				Player_GrantResearch (Player,"eldar_research_infantryhealth_1");
				Player_GrantResearch (Player,"eldar_research_infantryhealth_2");
				Player_GrantResearch (Player,"eldar_warlock_ability_research");
				Player_GrantResearch (Player,"eldar_warlock_ability_research_1");
				Player_GrantResearch (Player,"eldar_wargear01");
				Player_GrantResearch (Player,"eldar_wargear02");
				Player_GrantResearch (Player,"eldar_wargear03");
				Player_GrantResearch (Player,"eldar_wargear04");
				Player_GrantResearch (Player,"eldar_wargear05");
				Player_GrantResearch (Player,"eldar_wargear06");
				Player_GrantResearch (Player,"eldar_wargear07");
				Player_GrantResearch (Player,"eldar_wargear08");
				Player_GrantResearch (Player,"eldar_wargear09");
				Player_GrantResearch (Player,"eldar_wargear10");
				
			-- Imperial Guard	
			elseif (Player_GetRaceName(Player) == "guard_race") then
				blueprint_commander = "guard_squad_command_squad";
				blueprint_squad = "guard_squad_guardsmen";
				blueprint_builder_squad = "guard_squad_enginseer";
				Player_GrantResearch (Player,"guard_research_command_squad_size");
				Player_GrantResearch (Player,"guard_guardsman_morale");
				Player_GrantResearch (Player,"guard_guardsman_morale_2");
				Player_GrantResearch (Player,"guard_upgrade_guardsmen_health");
				Player_GrantResearch (Player,"guard_upgrade_guardsmen_range");
				Player_GrantResearch (Player,"guard_upgrade_weapon_specialization");
				Player_GrantResearch (Player,"guard_wargear01");
				Player_GrantResearch (Player,"guard_wargear02");
				Player_GrantResearch (Player,"guard_wargear03");
				Player_GrantResearch (Player,"guard_wargear04");
				Player_GrantResearch (Player,"guard_wargear05");
				Player_GrantResearch (Player,"guard_wargear06");
				Player_GrantResearch (Player,"guard_wargear07");
				Player_GrantResearch (Player,"guard_wargear08");
				Player_GrantResearch (Player,"guard_wargear09");
				Player_GrantResearch (Player,"guard_wargear10");
				
			-- Inquisition Daemonhunters
			elseif (Player_GetRaceName(Player) == "inquisition_daemonhunt_race") then
				blueprint_commander = "inquisition_squad_inquisitor_lord";
				blueprint_squad = "inquisition_squad_grey_knights_terminator";
				blueprint_builder_squad = "inquisition_squad_archivist";
				Player_GrantResearch (Player,"inquisition_medikits");
				Player_GrantResearch (Player,"inquisition_officers_weapons");
				Player_GrantResearch (Player,"inquisition_advanced_weapons");
				Player_GrantResearch (Player,"inquisition_advanced_weapons_2");
				Player_GrantResearch (Player,"inquisition_special_squads");
				Player_GrantResearch (Player,"inquisition_psy_power_1");
				Player_GrantResearch (Player,"inquisition_psy_power_2");
				Player_GrantResearch (Player,"inquisition_psy_power_3");
				Player_GrantResearch (Player,"inquisition_targeters");
				Player_GrantResearch (Player,"inquisition_temporal_power");
				
			-- Necrons	
			elseif (Player_GetRaceName(Player) == "necron_race") then
				blueprint_commander = "necron_lord_squad";
				blueprint_squad = "necron_basic_warrior_squad";
				blueprint_builder_squad = "necron_builder_scarab_squad";
				Player_GrantResearch (Player,"necron_phylactery_research");
				Player_GrantResearch (Player,"necron_resurrection_orb_research");
				Player_GrantResearch (Player,"necron_nightmare_shroud_research");
				Player_GrantResearch (Player,"necron_warrior_boost");
				Player_GrantResearch (Player,"necron_warrior_boost_2");
				Player_GrantResearch (Player,"necron_wargear01");
				Player_GrantResearch (Player,"necron_wargear02");
				Player_GrantResearch (Player,"necron_wargear03");
				Player_GrantResearch (Player,"necron_wargear04");
				Player_GrantResearch (Player,"necron_wargear05");
				Player_GrantResearch (Player,"necron_wargear06");
				Player_GrantResearch (Player,"necron_wargear07");
				Player_GrantResearch (Player,"necron_wargear09");
				Player_GrantResearch (Player,"necron_wargear10");
				
			-- Orks
			elseif (Player_GetRaceName(Player) == "ork_race") then
				blueprint_commander = "ork_squad_warboss";
				blueprint_squad = "ork_flash_gitz_squad";
				blueprint_builder_squad = "ork_squad_grot";
				Player_GrantResearch (Player,"ork_research_big_mek_1");
				Player_GrantResearch (Player,"ork_research_big_mek_2");
				Player_GrantResearch (Player,"ork_research_morechoppy");
				Player_GrantResearch (Player,"ork_research_evenmorechoppy");
				Player_GrantResearch (Player,"ork_research_tankbustabombs");
				Player_GrantResearch (Player,"ork_research_tougher_bosses");
				Player_GrantResearch (Player,"ork_research_tougher_bosses_2");
				Player_GrantResearch (Player,"ork_research_eavy_armor_boyz");
				Player_GrantResearch (Player,"ork_research_eavy_armor_boyz_2");
				Player_GrantResearch (Player,"ork_research_tougherorks");
				Player_GrantResearch (Player,"ork_research_tougherorks_2");
				Player_GrantResearch (Player,"ork_research_moredakka");
				Player_GrantResearch (Player,"ork_research_evenmoredakka");
				Player_GrantResearch (Player,"ork_research_eavy_armour");
				Player_GrantResearch (Player,"ork_research_eavy_armor_2");
				Player_GrantResearch (Player,"ork_wargear01");
				Player_GrantResearch (Player,"ork_wargear02");
				Player_GrantResearch (Player,"ork_wargear03");
				Player_GrantResearch (Player,"ork_wargear04");
				Player_GrantResearch (Player,"ork_wargear05");
				Player_GrantResearch (Player,"ork_wargear06");
				Player_GrantResearch (Player,"ork_wargear07");
				Player_GrantResearch (Player,"ork_wargear08");
				Player_GrantResearch (Player,"ork_wargear09");
				Player_GrantResearch (Player,"ork_wargear10");
				Player_SetResource(Player, RT_Pop,200);

			-- Space Marines
			elseif (Player_GetRaceName(Player) == "space_marine_race") then
				blueprint_commander = "space_marine_squad_force_commander";
				blueprint_squad = "space_marine_squad_terminator";
				blueprint_builder_squad = "space_marine_squad_servitor";
				Player_GrantResearch (Player,"marine_force_commander_research_2");
				Player_GrantResearch (Player,"marine_commander_health_research_1");
				Player_GrantResearch (Player,"marine_commander_health_research_2");
				Player_GrantResearch (Player,"marine_accuracy_upgrade_research");
				Player_GrantResearch (Player,"marine_accuracy_upgrade_research_2");
				Player_GrantResearch (Player,"marine_health_upgrade_research");
				Player_GrantResearch (Player,"marine_health_upgrade_research_2");
				Player_GrantResearch (Player,"marine_max_weapons_research");
				Player_GrantResearch (Player,"marine_frag_grenade_research");
				Player_GrantResearch (Player,"marine_sergeant_melee_upgrade_1");
				Player_GrantResearch (Player,"marine_sergeant_ranged_upgrade_1");
				Player_GrantResearch (Player,"marine_wargear01");
				Player_GrantResearch (Player,"marine_wargear02");
				Player_GrantResearch (Player,"marine_wargear03");
				Player_GrantResearch (Player,"marine_wargear04");
				Player_GrantResearch (Player,"marine_wargear05");
				Player_GrantResearch (Player,"marine_wargear06");
				Player_GrantResearch (Player,"marine_wargear07");
				Player_GrantResearch (Player,"marine_wargear09");
				Player_GrantResearch (Player,"marine_wargear10");
				
			-- Tau Empire	
			elseif (Player_GetRaceName(Player) == "tau_race") then
				blueprint_commander = "tau_commander_squad";
				blueprint_squad = "tau_fire_warrior_squad";
				blueprint_builder_squad = "tau_builder_squad";
				Player_GrantResearch (Player,"tau_wargear01");
				Player_GrantResearch (Player,"tau_wargear02");
				Player_GrantResearch (Player,"tau_wargear03");
				Player_GrantResearch (Player,"tau_wargear04");
				Player_GrantResearch (Player,"tau_wargear05");
				Player_GrantResearch (Player,"tau_wargear06");
				Player_GrantResearch (Player,"tau_wargear07");
				Player_GrantResearch (Player,"tau_wargear09");
				Player_GrantResearch (Player,"tau_wargear10");
				Player_GrantResearch (Player,"tau_targeting_optics");
				Player_GrantResearch (Player,"tau_improved_power_source_research");
				Player_GrantResearch (Player,"tau_improved_metallurgy");

			-- Tyranid Swarm	
			elseif (Player_GetRaceName(Player) == "tyranids_race") then
				blueprint_commander = "tyranids_squad_hive_tyrant_survival";
				blueprint_squad = "tyranids_squad_warrior_survival";
				blueprint_builder_squad = "tyranids_squad_woth";
				Player_GrantResearch (Player,"tyranids_hivetyrant_implant_research");
				Player_GrantResearch (Player,"tyranids_hivetyrant_lwhip_bsword_research");
				Player_GrantResearch (Player,"tyranids_hivetyrant_toxic_miasma_research");
				Player_GrantResearch (Player,"tyranids_hivetyrant_warp_field_research");
				Player_GrantResearch (Player,"tyranids_warrior_adrenal_glands_research");
				Player_GrantResearch (Player,"tyranids_warrior_bioplasma_research");
				Player_GrantResearch (Player,"tyranids_warrior_ext_carapace_research");
				Player_GrantResearch (Player,"tyranids_warrior_leaping_research"); 
				
			-- Witch Hunters	
			elseif (Player_GetRaceName(Player) == "witch_hunters_race") then
				blueprint_commander = "witch_hunters_squad_canoness_survival";
				blueprint_squad = "witch_hunters_squad_battle_sister";
				blueprint_builder_squad = "witch_hunters_squad_sentinel_builder";
				Player_GrantResearch (Player,"witch_hunters_research_actoffaith_emperor_light");
				Player_GrantResearch (Player,"witch_hunters_research_actoffaith_hand_emperor");
				Player_GrantResearch (Player,"witch_hunters_research_actoffaith_spirit_martyr");
				Player_GrantResearch (Player,"witch_hunters_research_adepta_sororitas_frag_grenade");
				Player_GrantResearch (Player,"witch_hunters_research_adepta_sororitas_melta_bomb");
				Player_GrantResearch (Player,"witch_hunters_research_canoness_jetpack");
				Player_GrantResearch (Player,"witch_hunters_research_canoness_liber_heresius");
				Player_GrantResearch (Player,"witch_hunters_research_canoness_mantle_orphelia");
				Player_GrantResearch (Player,"witch_hunters_research_canoness_psychic_hood");
				Player_GrantResearch (Player,"witch_hunters_research_celestian_holy_hatred");
				Player_GrantResearch (Player,"witch_hunters_research_female_inquisitor_exagrammic_wards");
				Player_GrantResearch (Player,"witch_hunters_research_female_inquisitor_scourging");
				Player_GrantResearch (Player,"witch_hunters_research_sororitas_dominion");
				Player_GrantResearch (Player,"witch_hunters_research_sororitas_maximum_health");
				Player_GrantResearch (Player,"witch_hunters_shrine_faith_shield");
				
			end
			Modifier_ApplyToPlayer(reqmodifier, Player)
			Modifier_ApplyToPlayer(powermodifier, Player)
			Util_CreateSquadsAtPositionEx(Player, "sg_survivor_commander", blueprint_commander, Player_GetStartPosition(Player), 1, 1);
			for i = 1, 2 do
				Util_CreateSquadsAtPositionEx(Player, "sg_survivor"..i, blueprint_builder_squad, Player_GetStartPosition(Player), 1, 1);
				Util_CreateSquadsAtPositionEx(Player, "sg_survivor"..i, blueprint_squad, Player_GetStartPosition(Player), 1, 1);
			end
			
			g_survivor_count_total = 2;
			-- Set Survivor as opposite team of zombies.
			Player_SetTeam(Player, 0);
			
			g_DefaultSurvivor = Player;
		end
	end
	
end


function ZombieApocalypse_CheckZombies()
	for i = 1, g_zombiecount do
		if (SGroup_Exists("sg_zombie"..i) and SGroup_Count("sg_zombie"..i) < 1) then
			Util_CreateSquadsAtPosition(g_DefaultZombies, "sg_zombie"..i, "ork_squad_burnaz", Player_GetStartPosition(g_DefaultZombies), 1);
			if (Player1_IsZombie == true) then
				Util_CreateSquadsAtPosition(g_Player1, "sg_zombie"..i, "ork_squad_slugga", Player_GetStartPosition(g_Player1), 2);
			end
			if (Player2_IsZombie == true) then
				Util_CreateSquadsAtPosition(g_Player2, "sg_zombie"..i, "ork_squad_slugga", Player_GetStartPosition(g_Player2), 2);
				Util_CreateSquadsAtPosition(g_Player2, "sg_zombie"..i, "ork_squad_burnaz", Player_GetStartPosition(g_Player2), 1);
			end
			if (Player3_IsZombie == true) then
				Util_CreateSquadsAtPosition(g_Player3, "sg_zombie"..i, "ork_squad_slugga", Player_GetStartPosition(g_Player3), 2);
			end
			if (Player4_IsZombie == true) then
				Util_CreateSquadsAtPosition(g_Player4, "sg_zombie"..i, "ork_squad_slugga", Player_GetStartPosition(g_Player4), 2);
			end
			if (Player5_IsZombie == true) then
				Util_CreateSquadsAtPosition(g_Player5, "sg_zombie"..i, "ork_squad_slugga", Player_GetStartPosition(g_Player5), 2);
				Util_CreateSquadsAtPosition(g_Player5, "sg_zombie"..i, "ork_squad_burnaz", Player_GetStartPosition(g_Player5), 1);
			end
			if (Player6_IsZombie == true) then
				Util_CreateSquadsAtPosition(g_Player6, "sg_zombie"..i, "ork_squad_slugga", Player_GetStartPosition(g_Player6), 2);
			end
			if (Player7_IsZombie == true) then
				Util_CreateSquadsAtPosition(g_Player7, "sg_zombie"..i, "ork_squad_slugga", Player_GetStartPosition(g_Player7), 2);
				Util_CreateSquadsAtPosition(g_Player7, "sg_zombie"..i, "ork_squad_burnaz", Player_GetStartPosition(g_Player7), 1);
			end
			if (Player8_IsZombie == true) then
				Util_CreateSquadsAtPosition(g_Player8, "sg_zombie"..i, "ork_squad_slugga", Player_GetStartPosition(g_Player8), 2);
			end
			g_zombiecount = g_zombiecount + i_zombiecountdifficulty;
		end
	end
end

function ZombieApocalypse_CheckSquadsDead()

	for i_survivor_count = 1, 2 do
		if (SGroup_Count("sg_survivor"..i_survivor_count) < 1 and SGroup_Exists("sg_dummysquad"..i_survivor_count) == false) then
			SGroup_CreateIfNotFound("sg_dummysquad"..i_survivor_count);
			g_survivor_count = g_survivor_count - 1;
			if (g_survivor_count == 0) then
				if (SGroup_Exists("sg_survivor_commander") and SGroup_Count("sg_survivor_commander") < 1 and g_fc_died == 1) then
					Player_Kill(g_DefaultSurvivor)
					Util_CheckOneTeamLeft("zombieapocalypse")
				end
			end
		end
	end
	
	if (SGroup_Exists("ork_squad_warboss") and SGroup_Count("ork_squad_warboss") < 1) then
		Player_Kill(g_DefaultZombies)
		World_SetTeamWin(g_DefaultSurvivor, "assassinate")
		World_SetGameOver()
	end
	if (SGroup_Exists("sg_survivor_commander") and SGroup_Count("sg_survivor_commander") < 1 and g_fc_died == nil) then
		Util_MissionTitle("Your Force Commander died! No more reinforcements will come.")
		g_fc_died = 1;
	end

end

function ZombieApocalypse_ZombieTactics()

	g_zombietactic = World_GetRand(1,1000);
	
	for i = 1, g_zombiecount do
		if (SGroup_Exists("sg_zombie"..i)) then
			if (g_zombietactic >= 950) then
				Cmd_AttackMovePos("sg_zombie"..i, Player_GetStartPosition(g_DefaultZombies));
			elseif (g_zombietactic >= 900) then
				Cmd_StopSquads("sg_zombie"..i)
			elseif (g_zombietactic >= 800) then
				ZombieApocalypse_ZombieTactics_MoveTo();
				Cmd_AttackMovePos("sg_zombie"..i, Player_GetStartPosition(g_zombiemovement_position));
			else
				for ii = 1, 11 do
					if (SGroup_Exists("sg_survivor"..ii) and SGroup_Count("sg_survivor"..ii) >= 1) then
						Cmd_AttackMovePos("sg_zombie"..i, SGroup_GetPosition("sg_survivor"..ii));
					end
				end
			end
			g_zombietactic = World_GetRand(1,1000);
		end
	end
end

function ZombieApocalypse_ZombieTactics_MoveTo()
	
	g_zombiemovement_random = World_GetRand(1,800);
	
	if (Player1_IsZombie == true and g_zombietactic <= 100) then
		g_zombiemovement_position = g_Player1;
	elseif (Player2_IsZombie == true and g_zombietactic <= 200) then
		g_zombiemovement_position = g_Player2;
	elseif (Player3_IsZombie == true and g_zombietactic <= 300) then
		g_zombiemovement_position = g_Player3;
	elseif (Player4_IsZombie == true and g_zombietactic <= 400) then
		g_zombiemovement_position = g_Player4;
	elseif (Player5_IsZombie == true and g_zombietactic <= 500) then
		g_zombiemovement_position = g_Player5;
	elseif (Player6_IsZombie == true and g_zombietactic <= 600) then
		g_zombiemovement_position = g_Player6;
	elseif (Player7_IsZombie == true and g_zombietactic <= 700) then
		g_zombiemovement_position = g_Player7;
	elseif (Player8_IsZombie == true and g_zombietactic <= 800) then
		g_zombiemovement_position = g_Player8;
	else g_zombiemovement_position = g_DefaultZombies;
	end
end

function ZombieApocalypse_Reinforcements()
	
	if (SGroup_Exists("sg_survivor_commander") and SGroup_Count("sg_survivor_commander") >= 1) then
		g_survivor_count_total = g_survivor_count_total + 1;
		Util_CreateSquadsAtPositionEx(g_DefaultSurvivor, "sg_survivor"..g_survivor_count_total, blueprint_squad, SGroup_GetPosition("sg_survivor_commander"), 1, 1);
		World_FXEventSquad("data:Art/Events/Unit_Upgrade_Morale_FX/reinforce_marine_trooper", "sg_survivor_commander")
		local warning = ("Reinforcements arrived!")
		EventCue_DoEvent("reinforce", "taskbar_button_click", warning, warning)
		g_reinforcement = 1;
		g_survivor_count = g_survivor_count + 1;
	end
	
end

function ZombieApocalypse_MassiveWaves()

	if (Player1_IsZombie == true) then
		Util_CreateSquadsAtPosition(g_Player1, "sg_zombie"..g_zombiecount, "ork_squad_slugga", Player_GetStartPosition(g_Player1), 20);
		Util_CreateSquadsAtPosition(g_Player1, "sg_zombie"..g_zombiecount, "ork_squad_burnaz", Player_GetStartPosition(g_Player1), 5);
	end
	if (Player2_IsZombie == true) then
		Util_CreateSquadsAtPosition(g_Player2, "sg_zombie"..g_zombiecount, "ork_squad_ard_boy", Player_GetStartPosition(g_Player2), 10);
		Util_CreateSquadsAtPosition(g_Player2, "sg_zombie"..g_zombiecount, "ork_squad_burnaz", Player_GetStartPosition(g_Player2), 5);
		Util_CreateSquadsAtPosition(g_Player2, "sg_zombie"..g_zombiecount, "ork_squad_looted_tank", Player_GetStartPosition(g_Player2), 2);
	end
	if (Player3_IsZombie == true) then
		Util_CreateSquadsAtPosition(g_Player3, "sg_zombie"..g_zombiecount, "ork_squad_warboss", Player_GetStartPosition(g_Player3), 2);
	end
	if (Player4_IsZombie == true) then
		Util_CreateSquadsAtPosition(g_Player4, "sg_zombie"..g_zombiecount, "ork_squad_ard_boy", Player_GetStartPosition(g_Player4), 10);
	end
	if (Player5_IsZombie == true) then
		Util_CreateSquadsAtPosition(g_Player5, "sg_zombie"..g_zombiecount, "ork_squad_killa_kan", Player_GetStartPosition(g_Player5), 3);
		Util_CreateSquadsAtPosition(g_Player5, "sg_zombie"..g_zombiecount, "ork_squad_burnaz", Player_GetStartPosition(g_Player5), 5);
	end
	if (Player6_IsZombie == true) then
		Util_CreateSquadsAtPosition(g_Player6, "sg_zombie"..g_zombiecount, "ork_squad_slugga", Player_GetStartPosition(g_Player6), 10);
	end
	if (Player7_IsZombie == true) then
		Util_CreateSquadsAtPosition(g_Player7, "sg_zombie"..g_zombiecount, "ork_squad_slugga", Player_GetStartPosition(g_Player7), 10);
		Util_CreateSquadsAtPosition(g_Player7, "sg_zombie"..g_zombiecount, "ork_squad_weirdboy", Player_GetStartPosition(g_Player7), 2);
	end
	if (Player8_IsZombie == true and (Cpu_GetDifficulty(g_DefaultZombies) == AD_Insane)) then
		Util_CreateSquadsAtPosition(g_Player8, "sg_zombie"..g_zombiecount, "ork_squad_shoota_boy", Player_GetStartPosition(g_Player8), 8);
		Util_CreateSquadsAtPosition(g_Player8, "sg_zombie"..g_zombiecount, "ork_squad_burnaz", Player_GetStartPosition(g_Player8), 8);
	elseif (Player8_IsZombie == true) then
		Util_CreateSquadsAtPosition(g_Player8, "sg_zombie"..g_zombiecount, "ork_squad_slugga", Player_GetStartPosition(g_Player8), 5);
		Util_CreateSquadsAtPosition(g_Player8, "sg_zombie"..g_zombiecount, "ork_squad_burnaz", Player_GetStartPosition(g_Player8), 5);
	end
	Util_MissionTitle("You hear a distant WAAAAAGGH! Get ready...")
	Sound_Play("general_ping")
end

Scar_AddInit(ZombieApocalypse)
