#pragma once

#include "Classes.hpp"
#include "includes.hpp"
#include "Settings.hpp"
#include "native.hpp"

namespace Aimbot
{

	uintptr_t GetDistancePlayer()
	{
		uintptr_t ClosestPlayer = 0;
		float closestdist = FLT_MAX;
		float dist;
		hk_World* World = (hk_World*)*(uint64_t*)(FiveM::World);
		if (!World)
			return NULL;

		hk_Ped* LocalPlayer = World->LocalPlayer();
		if (!LocalPlayer)
			return NULL;

		hk_ReplayInterface* ReplayInterface = (hk_ReplayInterface*)*(uint64_t*)(FiveM::ReplayInterface);
		if (!ReplayInterface)
			return NULL;

		hk_PedInterface* PedInterface = ReplayInterface->PedInterface();
		if (!PedInterface)
			return NULL;

		for (int index = 0; index < PedInterface->PedMaximum(); index++)
		{
			hk_Ped* Peds = PedInterface->PedList()->Ped(index);
			if (!Peds) continue;

	
			if (Settings::Aimbot::TypeOfTarget == 1)
			{

				auto ped_type = Peds->GetPedType();
				if (!ped_type) continue;
				ped_type = ped_type << 11 >> 25;


				if (ped_type != 2) {
					continue;
				}
			}
			if (Settings::Aimbot::TypeOfTarget == 2)
			{
				auto ped_type = Peds->GetPedType();
				if (!ped_type) continue;

				ped_type = ped_type << 11 >> 25;


				if (ped_type == 2) {
					continue;
				}
			}

			if (Peds == LocalPlayer) continue;

			if (Peds->GetHealth() <= 0) continue;

			Vector3 DistanceCalculation = (LocalPlayer->GetCoordinate() - Peds->GetCoordinate());
			double Distance = sqrtf(DistanceCalculation.x * DistanceCalculation.x + DistanceCalculation.y * DistanceCalculation.y + DistanceCalculation.z * DistanceCalculation.z);







			if (Distance <= Settings::Aimbot::DistanceAimbotLimit)
			{
				ImVec2 PedPos = FiveM::WorldToScreen(Peds->GetCoordinate());

				ImVec2 Head = FiveM::GetBonePosW2S(reinterpret_cast<uint64_t>(Peds), 0x796e);
				if (!IsOnScreen(Head)) continue;

				ImVec2 middle = ImVec2(FiveM::WindowSize.x / 2 , FiveM::WindowSize.y / 2 );
				dist = FiveM::pythag(middle, Head);

				if (dist < closestdist)
				{
					{


						closestdist = dist;
						ClosestPlayer = reinterpret_cast<uint64_t>(Peds);


					}
				}

			}
		}
		return ClosestPlayer;
	}
	float get_distance(Vector3 pointA, Vector3 pointB) {
		float x_ba = (float)(pointB.x - pointA.x);
		float y_ba = (float)(pointB.y - pointA.y);
		float z_ba = (float)(pointB.z - pointA.z);
		float y_2 = y_ba * y_ba;
		float x_2 = x_ba * x_ba;
		float sum_2 = y_2 + x_2;
		return (float)sqrtf(sum_2 + z_ba);
	}
	
	PVector3 ToPVector3(Vector3 pVec)
	{
		return PVector3(pVec.x, pVec.y, pVec.z);
	}

	Vector3 ToVector3(PVector3 pVec)
	{
		return Vector3(pVec.x, pVec.y, pVec.z);
	}
	DWORD64 TICKS = GetTickCount64();


	void aimbot_silent(void)
	{

		auto player = PLAYER::PLAYER_PED_ID();
		auto WeaponHash = WEAPON::GET_SELECTED_PED_WEAPON(player);
		auto WeaponDamage = WEAPON::GET_WEAPON_DAMAGE(WeaponHash, NULL);
		
		if (Settings::Aimbot::Check_Fov)
		{
			auto get_distance = [](double x1, double y1, double x2, double y2) {
				return sqrtf(pow(x2 - x1, 2.0) + pow(y2 - y1, 2.0));
			};
			hk_Ped* PedCC = (hk_Ped*)GetDistancePlayer();
			if (!PedCC) return;
			ImVec2 screen = FiveM::WorldToScreen(PedCC->GetCoordinate());

			auto center_x = FiveM::WindowSize.x / 2;
			auto center_y = FiveM::WindowSize.y / 2;
			auto fov = get_distance(center_x, center_y, screen.x, screen.y);
			if (fov < Settings::Aimbot::AimbotFov)
				return;

		}


		if (GetDistancePlayer() != NULL && PED::IS_PED_SHOOTING(player))
		{

			if (!Settings::Aimbot::magic)
			{
				if (GetTickCount64() - TICKS > 450.f)
				{
				
					hk_Ped* PedC = (hk_Ped*)GetDistancePlayer();
					if (!PedC) return;
					auto InPut = ENTITY::GET_ENTITY_COORDS(player, (bool)true);

					auto Out = ToPVector3(PedC->GetCoordinate());
					MISC::SHOOT_SINGLE_BULLET_BETWEEN_COORDS(InPut.x, InPut.y, InPut.z, Out.x, Out.y, Out.z, (int)WeaponDamage, false, WeaponHash, player, true, false, 3.f);
				}

			}
			else
			{

				if (GetTickCount64() - TICKS > 450.f)
				{

					hk_Ped* PedC = (hk_Ped*)GetDistancePlayer();
					if (!PedC) return;

					auto Out = ToPVector3(PedC->GetCoordinate());
					MISC::SHOOT_SINGLE_BULLET_BETWEEN_COORDS(Out.x -0.1, Out.y, Out.z , Out.x + 0.1, Out.y, Out.z, (int)WeaponDamage, false, WeaponHash, player, true, false, 3.f);
				}
			}
			TICKS = GetTickCount64();


		}
	}

	void AimVec(Vector3 point)
	{
	//	auto draw_list = ImGui::GetBackgroundDrawList();
		DWORD64 addr = FiveM::GetCamera();
		if (addr)
		{
			Vector3 CrosshairPos = *(Vector3*)(addr + 0x60);
			Vector3 caca(CrosshairPos.x - point.x, CrosshairPos.y - point.y, CrosshairPos.z - point.z);
			float distance = caca.Length();

			__try
			{
				uint64_t CamData = *(DWORD64*)(addr + 0x10);
				if (CamData != NULL)
				{
					if (*(float*)(CamData + 0x130) == 8.0f)
					{
						*(float*)(CamData + 0x130) = 111.0f;
						*(float*)(CamData + 0x134) = 111.0f;
						*(float*)(CamData + 0x4CC) = 0.0f;

						if (*(float*)(CamData + 0x49C) == 1.0f)
							*(float*)(CamData + 0x49C) = 0.0f;

						*(float*)(CamData + 0x2AC) = 0.0f;
						*(float*)(CamData + 0x2B0) = 0.0f;
					}
				}
			}
			__except ((GetExceptionCode() == EXCEPTION_ACCESS_VIOLATION) ? EXCEPTION_EXECUTE_HANDLER : EXCEPTION_CONTINUE_SEARCH) {}



			int smooth = 0;

			smooth = Settings::Aimbot::AimbotSmooth + 1.5f;

			float threshold = 0.5f / (10 * 20);
			Vector3 Out = Vector3((point.x - CrosshairPos.x) / distance, (point.y - CrosshairPos.y) / distance, (point.z - CrosshairPos.z) / distance);

			if (smooth <= 1)
			{
				*(Vector3*)(addr + 0x40) = Out;  //FPS
				*(Vector3*)(addr + 0x3D0) = Out; //TPS
			}
			else
			{
				//First Person
				{
					Vector3 angles = *(Vector3*)(addr + 0x40);
					if (((Out.x - angles.x) > threshold) || ((angles.x - Out.x) > threshold))
					{
						if (angles.x > Out.x)
							*(float*)(addr + 0x40 + 0x0) -= threshold;
						else if (angles.x < Out.x)
							*(float*)(addr + 0x40 + 0x0) += threshold;
					}
					if (((Out.y - angles.y) > threshold) || ((angles.y - Out.y) > threshold))
					{
						if (angles.y > Out.y)
							*(float*)(addr + 0x40 + 0x4) -= threshold;
						else if (angles.y < Out.y)
							*(float*)(addr + 0x40 + 0x4) += threshold;
					}
					if (((Out.z - angles.z) > threshold) || ((angles.z - Out.z) > threshold))
					{
						if (angles.z > Out.z)
							*(float*)(addr + 0x40 + 0x8) -= threshold;
						else if (angles.z < Out.z)
							*(float*)(addr + 0x40 + 0x8) += threshold;
					}
				}
				//Third Person
				{
					Vector3 angles = *(Vector3*)(addr + 0x3D0);
					if (((Out.x - angles.x) > threshold) || ((angles.x - Out.x) > threshold))
					{
						if (angles.x > Out.x)
							*(float*)(addr + 0x3D0 + 0x0) -= threshold;
						else if (angles.x < Out.x)
							*(float*)(addr + 0x3D0 + 0x0) += threshold;
					}
					if (((Out.y - angles.y) > threshold) || ((angles.y - Out.y) > threshold))
					{
						if (angles.y > Out.y)
							*(float*)(addr + 0x3D0 + 0x4) -= threshold;
						else if (angles.y < Out.y)
							*(float*)(addr + 0x3D0 + 0x4) += threshold;
					}
					if (((Out.z - angles.z) > threshold) || ((angles.z - Out.z) > threshold))
					{
						if (angles.z > Out.z)
							*(float*)(addr + 0x3D0 + 0x8) -= threshold;
						else if (angles.z < Out.z)
							*(float*)(addr + 0x3D0 + 0x8) += threshold;
					}
				}
			}
		}
	}
	void do_aimbot(uintptr_t entity)
	{ // pretty buggy, needs playing around with sensitivity

		auto get_distance = [](double x1, double y1, double x2, double y2) {
			return sqrtf(pow(x2 - x1, 2.0) + pow(y2 - y1, 2.0));
		};

		auto bone_pos = FiveM::GetBonePosW2S(entity, 0x796e);
		//Vec3 bone_postest = GetBonePos(entity, SKEL_Head);
		auto center_x = FiveM::WindowSize.x / 2;
		auto center_y = FiveM::WindowSize.y / 2;
		if (bone_pos.x < 0 || bone_pos.x > FiveM::WindowSize.x || bone_pos.y < 0 || bone_pos.y > FiveM::WindowSize.y) return;

		float aimspeedon = Settings::Aimbot::AimbotSmooth + 1.5f;


		switch (Settings::Aimbot::AimbotBone)
		{
		case 0:
			//bone_pos = FiveM::GetBonePosW2S(entity, SKEL_Head);
			break;
		case 1:
			bone_pos = FiveM::GetBonePosW2S(entity, 0x60F2);
			break;
		case 2:
			bone_pos = FiveM::GetBonePosW2S(entity, 0xF9BB);
			break;
		case 3:
			bone_pos = FiveM::GetBonePosW2S(entity, 0x9000);
			break;
		case 4:
			bone_pos = FiveM::GetBonePosW2S(entity, 0x9D4D);
			break;
		case 5:
			bone_pos = FiveM::GetBonePosW2S(entity, 0xb1);
			break;
		}



		ImVec2 screen = bone_pos;

		auto fov = get_distance(center_x, center_y, screen.x, screen.y);
		float TargetX = 0;
		float TargetY = 0;

		if (screen.x != 0) {
			if (screen.x > center_x) {
				TargetX = -(center_x - screen.x);
				TargetX /= aimspeedon + 1.4f;
				if (TargetX + center_x > center_x * 2) TargetX = 0;
			}

			if (screen.x < center_x) {
				TargetX = screen.x - center_x;
				TargetX /= aimspeedon + 1.4f;
				if (TargetX + center_x < 0) TargetX = 0;
			}
		}

		if (screen.y != 0) {
			if (screen.y > center_y) {
				TargetY = -(center_y - screen.y);
				TargetY /= aimspeedon + 1.4f;
				if (TargetY + center_y > center_y * 2) TargetY = 0;
			}

			if (screen.y < center_y) {
				TargetY = screen.y - center_y;
				TargetY /= aimspeedon + 1.4f;
				if (TargetY + center_y < 0) TargetY = 0;
			}
		}


		/*
		float theNum = floor(TargetX / Settings::Aimbot::AimbotSmooth);
		float result = theNum / 6.666666666666667f;

		float theNum1 = floor(TargetY / Settings::Aimbot::AimbotSmooth);
		float resulte = theNum1 / 6.666666666666667f;
		float result1 = -(resulte);
		*/


		if (fov < Settings::Aimbot::AimbotFov)
		{

	
				mouse_event(MOUSEEVENTF_MOVE, static_cast<DWORD>(TargetX), static_cast<DWORD>(TargetY), 0, 0);

			


			
		}



	}

	void Hook(void)
	{
	

		if (Settings::Aimbot::Aimbot)
		{

			

			if (SAFE_CALL(GetAsyncKeyState)(Settings::Aimbot::Hotkey) & 0x8000) {

				if (Settings::Aimbot::AimbotTypes == 0)
				{
			uintptr_t pCPed = GetDistancePlayer();
			if (!pCPed) return;
			do_aimbot(pCPed);

				}
				else
				{
					auto get_distance = [](double x1, double y1, double x2, double y2) {
						return sqrtf(pow(x2 - x1, 2.0) + pow(y2 - y1, 2.0));
					};

					auto center_x = FiveM::WindowSize.x / 2;
					auto center_y = FiveM::WindowSize.y / 2;
					


					uintptr_t entity = GetDistancePlayer();
					if (!entity) return;
					auto bone_pos = FiveM::GetBonePos(entity, 0x796e);

	

					switch (Settings::Aimbot::AimbotBone)
					{
					case 0:
						//bone_pos = FiveM::GetBonePosW2S(entity, SKEL_Head);
						break;
					case 1:
						bone_pos = FiveM::GetBonePos(entity, 0x60F2);
						break;
					case 2:
						bone_pos = FiveM::GetBonePos(entity, 0xF9BB);
						break;
					case 3:
						bone_pos = FiveM::GetBonePos(entity, 0x9000);
						break;
					case 4:
						bone_pos = FiveM::GetBonePos(entity, 0x9D4D);
						break;
					case 5:
						bone_pos = FiveM::GetBonePos(entity, 0xb1);
						break;
					}
					ImVec2 screen = FiveM::WorldToScreen(bone_pos);

					auto fov = get_distance(center_x, center_y, screen.x, screen.y);

					if (fov < Settings::Aimbot::AimbotFov)
					{
//						if (bone_pos.x < 0 || bone_pos.x > FiveM::WindowSize.x || bone_pos.y < 0 || bone_pos.y > FiveM::WindowSize.y) return;


						AimVec(bone_pos);
					}
				}
		

			}



		}
	

	}
}#pragma once

#include "includes.hpp"

struct Vector3Fix
{
public:
	Vector3Fix() = default;

	Vector3Fix(float x, float y, float z) :
		x(x), y(y), z(z)
	{}
public:
	float x{};
private:
	char m_padding1[0x04]{};
public:
	float y{};
private:
	char m_padding2[0x04]{};
public:
	float z{};
private:
	char m_padding3[0x04]{};
};


class Vector3 final
{
public:

    float x, y, z;

    Vector3(const float x, const float y, const float z) : x(x), y(y), z(z) {}
    Vector3 operator + (const Vector3& rhs) const { return Vector3(x + rhs.x, y + rhs.y, z + rhs.z); }
    Vector3 operator - (const Vector3& rhs) const { return Vector3(x - rhs.x, y - rhs.y, z - rhs.z); }
    Vector3 operator * (const float& rhs) const { return Vector3(x * rhs, y * rhs, z * rhs); }
    Vector3 operator / (const float& rhs) const { return Vector3(x / rhs, y / rhs, z / rhs); }
    bool operator == (const Vector3& rhs) const { return x == rhs.x && y == rhs.y && z == rhs.z; }
    Vector3& operator += (const Vector3& rhs) { return *this = *this + rhs; }
    Vector3& operator -= (const Vector3& rhs) { return *this = *this - rhs; }
    Vector3& operator *= (const float& rhs) { return *this = *this * rhs; }
    Vector3& operator /= (const float& rhs) { return *this = *this / rhs; }
    float Length() const { return sqrt(x * x + y * y + z * z); }
    Vector3 Normalize() const { return *this * (1 / Length()); }
    float Distance(const Vector3& rhs) const { return (*this - rhs).Length(); }
    void Invert() { *this *= -1; }
    static Vector3 FromM128(__m128 in) { return Vector3(in.m128_f32[0], in.m128_f32[1], in.m128_f32[2]); }
}; 

class Vector4
{
public:
	float x, y, z, w;
};

inline ImVec2 GetWindowSize()
{
    RECT rect;
    HWND hwnd = GetActiveWindow();
    if (GetWindowRect(hwnd, &rect))
    {
        int width = rect.right - rect.left;
        int height = rect.bottom - rect.top;
        return ImVec2(width, height);
      
    }
    return ImVec2(0, 0);
}


namespace FiveM
{

    using namespace FiveM;
    inline DWORD64 flypatt;
    inline uint64_t World, ReplayInterface, W2S, BonePos, Camera , Waypoint;
    inline bool IsOnFiveM;

    inline DWORD Armor, EntityType, WeaponManager, PlayerInfo, Recoil, Spread , AmmoType ,WeaponName, AmmoExplosiveType , Range , ReloadMultiplier , VehiculeReloadMultiplier , IsInAVehicule;

    inline ImVec2 WindowSize = ImVec2(GetSystemMetrics(SM_CXSCREEN), GetSystemMetrics(SM_CYSCREEN));

	inline DWORD64 GetCamera()
	{
		if (Camera)
			return *(DWORD64*)(Camera + 0x0);
	}



    inline ImVec2 WorldToScreen(Vector3 pos)
    {
        auto& io = ImGui::GetIO();
        ImVec2 tempVec2;
        reinterpret_cast<bool(__fastcall*)(Vector3*, float*, float*)>(W2S)(&pos, &tempVec2.x, &tempVec2.y);
        tempVec2.x *= io.DisplaySize.x;
        tempVec2.y *= io.DisplaySize.y;
        return tempVec2;
    }
    inline float pythag(ImVec2 src, ImVec2 dst)
    {
        return sqrtf(pow(src.x - dst.x, 2) + pow(src.y - dst.y, 2));
    }

    inline float pythagVec3(Vector3 src, Vector3 dst)
    {
        return sqrtf(pow(src.x - dst.x, 2) + pow(src.y - dst.y, 2) + pow(src.z - dst.z, 2));
    }
	inline Vector3 GetBonePos(const uint64_t cPed, const int32_t wMask)
	{
		__m128 tempVec4;
		reinterpret_cast<void* (__fastcall*)(uint64_t, __m128*, int32_t)>(BonePos)(cPed, &tempVec4, wMask);
		return Vector3::FromM128(tempVec4);
	}
    inline ImVec2 GetBonePosW2S(const uint64_t cPed, const int32_t wMask)
    {
        __m128 tempVec4;
        reinterpret_cast<void* (__fastcall*)(uint64_t, __m128*, int32_t)>(BonePos)(cPed, &tempVec4, wMask);
        return WorldToScreen(Vector3::FromM128(tempVec4));
    }
}

inline bool IsOnScreen(ImVec2 coords)
{
    if (coords.x < 0.1f || coords.y < 0.1 || coords.x > ImGui::GetIO().DisplaySize.x || coords.y > ImGui::GetIO().DisplaySize.y)
    {
        return false;
    }
    else {
        return true;
    }
}
inline void DrawHealthBar(ImVec2 pos, ImVec2 dim, ImColor col)
{
    if (IsOnScreen(pos))
    {

        ImGui::GetBackgroundDrawList()->AddLine(pos, ImVec2(pos.x, pos.y - dim.y), col, dim.x);
    }
}
inline const char* Get_player_name(__int64 num)
{
	using GetName_t = const char* (*)(__int64 num);
	static auto GetName = (GetName_t)(Memory::PatternScan(E("40 53 48 83 EC ? 80 3D ? ? ? ? ? 8B D9 74 ? 33 D2"), NULL, NULL));	
	//	(CustomAPII::ScanSignature(NULL, "40 53 48 83 EC ? 80 3D ? ? ? ? ? 8B D9 74 ? 33 D2"));
		return GetName(num);

}
inline const char* get_weapon_name(DWORD hash)
{
	//removed xoring, cba to do properly.
	const char* dagger = ("Dagger");
	const char* bat = ("Bat");
	const char* bottle = ("Bottle");
	const char* crowbar = ("Crow Bar");
	const char* unarmed = ("None");
	const char* flashlight = ("Flash Light");
	const char* golfclub = ("Golf club");
	const char* hammer = ("Hammer");
	const char* hatchet = ("Hatchet");
	const char* knuckle = ("Knuckle");
	const char* knife = ("Knife");
	const char* machete = ("Machete");
	const char* switchblade = ("Switch Blade");
	const char* nightstick = ("Night Stick");
	const char* wrench = ("Wrench");
	const char* battleaxe = ("Battle Axe");
	const char* poolcue = ("Pool Cue");
	const char* pistol = ("Pistol");
	const char* pistolmk2 = ("Pistol MK2");
	const char* combatpistol = ("Combat Pistol");
	const char* appistol = ("AP Pistol");
	const char* stungun = ("Stungun");
	const char* pistol50 = ("Pistol 50");
	const char* snspistol = ("SNS PISTOL");
	const char* snspistolmk2 = ("SNS Pistol MK2");
	const char* heavypistol = ("Heavy Pistol");
	const char* vintagepistol = ("Vintage Pisol");
	const char* flaregun = ("Flare Gun");
	const char* marksmanpistol = ("marksmanpistol");
	const char* revolver = ("Revolver");
	const char* revolvermk2 = ("Revolver MK2");
	static auto doubleaction = ("Double Action");
	static auto microsmg = ("Micro Smg");
	static auto smg = ("Smg");
	static auto smgmk2 = ("Smg MK2");
	static auto assaultsmg = ("Assault Smg");
	static auto combatpdw = ("Combat PDW");
	static auto machinepistol = ("Machine Pistol");
	static auto minismg = ("Mini Smg");
	static auto pumpshotgun = ("Pump Shotgun");
	static auto pumpshotgun_mk2 = ("Pump Shotgun MK2");
	static auto sawnoffshotgun = ("Sawnoff Shotgun");
	static auto assaultshotgun = ("Sssault Shotgun");
	static auto bullpupshotgun = ("Bullpup Shotgun");
	static auto musket = ("Musket");
	static auto heavyshotgun = ("Heavy Shotgun");
	static auto dbshotgun = ("DB Shotgun");
	static auto autoshotgun = ("Auto Shotgun");
	static auto assaultrifle = ("Assault Rifle");
	static auto assaultrifle_mk2 = ("Assault Rifle MK2");
	static auto carbinerifle = ("Carbine Rifle");
	static auto carbinerifle_mk2 = ("Carbine Rifle MK2");
	static auto advancedrifle = ("Advanced Rifle");
	static auto specialcarbine = ("Special Carbine");
	static auto specialcarbine_mk2 = ("Special Carbine MK2");
	static auto bullpuprifle = ("Bullpup Rifle");
	static auto bullpuprifle_mk2 = ("Bullpup Rifle MK2");
	static auto compactrifle = ("Compact Rifle");
	static auto machine_gun = ("Machine Gun");
	static auto combatmg = ("Combat MG");
	static auto combatmg_mk2 = ("Combat MG MK2");
	static auto gusenberg = ("GUSENBERG");
	static auto sniperrifle = ("Sniper Rifle");
	static auto heavysniper = ("AWP");
	static auto heavysniper_mk2 = ("AWP MK2");
	static auto marksmanrifle = ("Marksman Rifle");
	static auto marksmanrifle_mk2 = ("Marksman Rifle MK2");
	static auto rpg = ("RPG");
	static auto grenadelauncher = ("Grenade Launcher");
	static auto grenadelauncher_smoke = ("Grenade Launcher Smoke");
	static auto minigun = ("MiniGun");
	static auto firework = ("FireWork");
	static auto railgun = ("RailGun");
	static auto hominglauncher = ("Homing Launcher");
	static auto compactlauncher = ("Compact Launcher");
	static auto grenade = ("Grenade");
	static auto bzgas = ("BZGAS");
	static auto smokegrenade = ("Smoke Grenade");
	static auto flare = ("Flare");
	static auto molotov = ("Molotov");
	static auto stickybomb = ("Sticky BOMB");
	static auto proxmine = ("Prox Mine");
	static auto snowball = ("SnowBall");
	static auto pipebomb = ("Pipe Bomb");
	static auto ball = ("Ball");
	static auto petrolcan = ("Petrol Can");
	static auto fireextinguisher = ("Fire Extinguisher");
	static auto parachute = ("Parachute");

	switch (hash)
	{
	case 0x92A27487:
		return dagger; break;
	case 0x958A4A8F:
		return bat; break;
	case 0xF9E6AA4B:
		return bottle; break;
	case 0x84BD7BFD:
		return crowbar; break;
	case 0xA2719263:
		return unarmed; break;
	case 0x8BB05FD7:
		return flashlight; break;
	case 0x440E4788:
		return golfclub; break;
	case 0x4E875F73:
		return hammer; break;
	case 0xF9DCBF2D:
		return hatchet; break;
	case 0xD8DF3C3C:
		return knuckle; break;
	case 0x99B507EA:
		return knife; break;
	case 0xDD5DF8D9:
		return machete; break;
	case 0xDFE37640:
		return switchblade; break;
	case 0x678B81B1:
		return nightstick; break;
	case 0x19044EE0:
		return wrench; break;
	case 0xCD274149:
		return battleaxe; break;
	case 0x94117305:
		return poolcue; break;
	case 0x1B06D571:
		return pistol; break;
	case 0xBFE256D4:
		return pistolmk2; break;
	case 0x5EF9FEC4:
		return combatpistol; break;
	case 0x22D8FE39:
		return appistol; break;
	case 0x3656C8C1:
		return stungun; break;
	case 0x99AEEB3B:
		return pistol50; break;
	case 0xBFD21232:
		return snspistol; break;
	case 0x88374054:
		return snspistolmk2; break;
	case 0xD205520E:
		return heavypistol; break;
	case 0x83839C4:
		return vintagepistol; break;
	case 0x47757124:
		return flaregun; break;
	case 0xDC4DB296:
		return marksmanpistol; break;
	case 0xC1B3C3D1:
		return revolver; break;
	case 0xCB96392F:
		return revolvermk2; break;
	case 0x97EA20B8:
		return doubleaction; break;
	case 0x13532244:
		return microsmg; break;
	case 0x2BE6766B:
		return smg; break;
	case 0x78A97CD0:
		return smgmk2; break;
	case 0xEFE7E2DF:
		return assaultsmg; break;
	case 0xA3D4D34:
		return combatpdw; break;
	case 0xDB1AA450:
		return machinepistol; break;
	case 0xBD248B55:
		return minismg; break;
	case 0x1D073A89:
		return pumpshotgun; break;
	case 0x555AF99A:
		return pumpshotgun_mk2; break;
	case 0x7846A318:
		return sawnoffshotgun; break;
	case 0xE284C527:
		return assaultshotgun; break;
	case 0x9D61E50F:
		return bullpupshotgun; break;
	case 0xA89CB99E:
		return musket; break;
	case 0x3AABBBAA:
		return heavyshotgun; break;
	case 0xEF951FBB:
		return dbshotgun; break;
	case 0x12E82D3D:
		return autoshotgun; break;
	case 0xBFEFFF6D:
		return assaultrifle; break;
	case 0x394F415C:
		return assaultrifle_mk2; break;
	case 0x83BF0278:
		return carbinerifle; break;
	case 0xFAD1F1C9:
		return carbinerifle_mk2; break;
	case 0xAF113F99:
		return advancedrifle; break;
	case 0xC0A3098D:
		return specialcarbine; break;
	case 0x969C3D67:
		return specialcarbine_mk2; break;
	case 0x7F229F94:
		return bullpuprifle; break;
	case 0x84D6FAFD:
		return bullpuprifle_mk2; break;
	case 0x624FE830:
		return compactrifle; break;
	case 0x9D07F764:
		return machine_gun; break;
	case 0x7FD62962:
		return combatmg; break;
	case 0xDBBD7280:
		return combatmg_mk2; break;
	case 0x61012683:
		return gusenberg; break;
	case 0x5FC3C11:
		return sniperrifle; break;
	case 0xC472FE2:
		return heavysniper; break;
	case 0xA914799:
		return heavysniper_mk2; break;
	case 0xC734385A:
		return marksmanrifle; break;
	case 0x6A6C02E0:
		return marksmanrifle_mk2; break;
	case 0xB1CA77B1:
		return rpg; break;
	case 0xA284510B:
		return grenadelauncher; break;
	case 0x4DD2DC56:
		return grenadelauncher_smoke; break;
	case 0x42BF8A85:
		return minigun; break;
	case 0x7F7497E5:
		return firework; break;
	case 0x6D544C99:
		return railgun; break;
	case 0x63AB0442:
		return hominglauncher; break;
	case 0x781FE4A:
		return compactlauncher; break;
	case 0x93E220BD:
		return grenade; break;
	case 0xA0973D5E:
		return bzgas; break;
	case 0xFDBC8A50:
		return smokegrenade; break;
	case 0x497FACC3:
		return flare; break;
	case 0x24B17070:
		return molotov; break;
	case 0x2C3731D9:
		return stickybomb; break;
	case 0xAB564B93:
		return proxmine; break;
	case 0x787F0BB:
		return snowball; break;
	case 0xBA45E8B8:
		return pipebomb; break;
	case 0x23C9F95C:
		return ball; break;
	case 0x34A67B97:
		return petrolcan; break;
	case 0x60EC506:
		return fireextinguisher; break;
	case 0xFBAB5776:
		return parachute; break;
	default:
		return unarmed; break;
	}
}

class  hk_FixedAmmoCount
{
public:


    float SetAmmo(float caca)
    {
        if (!this) return 0;
        return *(uint32_t*)(this + 0x18) = caca;
    }


};

class hk_AmmoCount
{
public:

    hk_FixedAmmoCount* FixedAmmoCount()
    {
        if (!this) return 0;
        return (hk_FixedAmmoCount*)(*(uint64_t*)(this + 0x0));
    }



};
class hk_AmmoInfo
{
public:


    hk_AmmoCount* AmmoCount()
    {
        if (!this) return 0;
        return (hk_AmmoCount*)(*(uint64_t*)(this + 0x8));
    }



};

class hk_WeaponInfo
{
public:
	uint64_t GetHash()
	{
		if (!this) return NULL;
		return *(uint64_t*)(this + 0x10);
	}

	uint64_t SetHash(DWORD caca)
	{
		if (!this) return NULL;
		return *(uint64_t*)(this + 0x10) = caca;
	}
    float SetSpread(float value)
    {
        if (!this) return 0;
        return *(float*)(this + FiveM::Spread) = value;
    }

    float SetRecoil(float value)
    {
        if (!this) return 0;
        return *(float*)(this + FiveM::Recoil) = value;
    }
    hk_AmmoInfo* AmmoInfo()
    {
        if (!this) return 0;
        return (hk_AmmoInfo*)(*(uint64_t*)(this + 0x60));
    }

    float SetReload(float value)
    {
        if (!this) return 0;
        return *(float*)(this + FiveM::ReloadMultiplier) = value;
    }
	float SetRange(float value)
	{
		if (!this) return 0;
		return *(float*)(this + FiveM::Range) = value;
	}
    int32_t  SetAmmoType(float value)
    {
        if (!this) return 0;
        return *(int32_t*)(this + FiveM::AmmoType) = value;
    }
    int32_t  SetAmmoExplosiveType(float value)
    {
        if (!this) return 0;
        return *(int32_t*)(this + FiveM::AmmoExplosiveType) = value;
    }
    


};
class hk_WeaponManager
{
public:
    hk_WeaponInfo* WeaponInfo()
    {
        if (!this) return 0;
        return (hk_WeaponInfo*)(*(uint64_t*)(this + 0x20));
    }

	Vector3 GetWeaponCoordinate()
	{
		if (!this) return Vector3{ 0,0,0 };
		return *(Vector3*)(this + 0x1A0);
	}
	Vector3 SetWeaponCoordinate(Vector3 Cords)
	{
		if (!this) return Vector3{ 0,0,0 };
		return *(Vector3*)(this + 0x1A0) = Cords;
	}

};

class hk_ObjectNavigationPed
{
public:
    Vector3 GetCoordinate()
    {
        if (!this) return Vector3{ 0,0,0 };
        return *(Vector3*)(this + 0x50);
    }
    Vector3 SetCoordinate(Vector3 Cords)
    {
        if (!this) return Vector3{ 0,0,0 };
        return *(Vector3*)(this + 0x50) = Cords;
    }

	Vector4 SetRotation(Vector4 Coords)
	{
		if (!this) return Vector4{ 0,0,0,0 };
		return *(Vector4*)(this + 0x30) = Coords;
	}

};
class hk_Vehicle
{
public:

	bool Godmode(bool value)
	{
		if (!this) return 0;
		return *(bool*)(this + 0x189) = value;
	}
	float GetMaxHealth()
	{
		if (!this) return 0;
		return *(float*)(this + 0x284);
	}
	float SetHealth(float Health)
	{
		if (!this) return 0;
		return *(float*)(this + 0x280) = Health;
	}

};

class hk_Gravity
{
public:

     bool IsNoGravity()
	{

		 if (*(uint16_t*)(this + 0x1A) == 780)
		 {
			 return true;
		 }
		 else
		 {
			 return false;
		 }

	}

	uint16_t SetNoGravity(bool caca)
	{
		if (!this) return 0;
		if (caca)
			return *(uint16_t*)(this + 0x1A) = 0x30C;
		else
		return *(uint16_t*)(this + 0x1A) = 0x304;
	}

};

class hk_Ped
{
public:
    hk_WeaponManager* WeaponManager()
    {
        if (!this) return 0;
        return (hk_WeaponManager*)(*(uint64_t*)(this + FiveM::WeaponManager ));
    }
	hk_Gravity* GravityManager()
	{
		if (!this) return 0;
		return (hk_Gravity*)(*(uintptr_t*)(this + 0x1110));
	}
	hk_Vehicle* VehicleManager()
	{
		if (!this) return 0;
		return (hk_Vehicle*)(*(uint64_t*)(this + 0xD30));
	}
    hk_ObjectNavigationPed* ObjectNavigation()
    {
        if (!this) return 0;
        return (hk_ObjectNavigationPed*)(*(uint64_t*)(this + 0x30));
    }
	bool IsInAVehicule()
	{
		if (!this) return false;
		if (*(BYTE*)(this + FiveM::IsInAVehicule) == 0x40)
		{
			return true;
		}
		else
		{
			return false;
		}
	}
	bool SetFreeze(bool toggle)
	{

		if (!this) return 0;
		return *(BYTE*)(this + 0x218) = toggle;
	}

	uint32_t SetSuperJump(bool toggle)
	{
		if (toggle)
		{
			if (!this) return 0;
			return *(BYTE*)(this + 0x218) |= 1 << 14;;
		}

	
	}



    Vector3 GetCoordinate()
    {
        if (!this) return Vector3{ 0,0,0 };
        return *(Vector3*)(this + 0x90);
    }
    Vector3 SetCoordinate(Vector3 Cords)
    {
        if (!this) return Vector3{ 0,0,0 };
        return *(Vector3*)(this + 0x90) = Cords;
    }

	Vector3 SetVelocity()
	{
		if (!this) return Vector3{ 0,0,0 };
		return *(Vector3*)(this + 0x320) = Vector3(0,0,0);
	}
    float GetHealth()
    {
        if (!this) return 0;
        return *(float*)(this + 0x280);
    } 
    float GetArmor()
    {
        if (!this) return 0;
        return *(float*)(this + FiveM::Armor);
    }
    float GetMaxHealth()
    {
        if (!this) return 0;
        return *(float*)(this + 0x284);
    }
    float SetHealth(float Health)
    {
        if (!this) return 0;
        return *(float*)(this + 0x280) = Health;
    }
    float SetArmor(float Armor)
    {
        if (!this) return 0;
        return *(float*)(this + FiveM::Armor) = Armor;
    }
    float SetMaxHealth()
    {
        if (!this) return 0;
        return *(float*)(this + 0x280) = GetMaxHealth();
    }
    uint32_t GetPedType()
    {
        if (!this) return 0;
        return *(uint32_t*)(this + FiveM::EntityType);
    }
	bool IsPedOrFalse()
	{
		if (!this) return 0;
		auto ped_type = this->GetPedType();

		ped_type = ped_type << 11 >> 25;
	
		if (ped_type != 2)
		{
			return true;

		}
		else
			return false;
	}
	BYTE SetInvisible(BYTE caca)
	{
		if (!this) return 0;
		return *(BYTE*)(this + 0x2C) = caca; // 0X1 ou 0X37
	}

	unsigned char Set_Ragdoll(bool value)
	{
		if (!this) return 0;
			if (value)
			{
				return *(unsigned char*)(this + 0x10B8) = 1;
			}
			else
			{
				return *(unsigned char*)(this + 0x10B8) = 32;
			}
		
	}


    
};


class hk_PedList
{
public:
    hk_Ped* Ped(int index)
    {
        if (!this) return 0;
        return (hk_Ped*)(*(uint64_t*)(this + (index * 0x10)));
    }
};



class hk_PedInterface
{
public:
    uint64_t PedMaximum()
    {
        if (!this) return 0;
        return (uint64_t)(*(uint64_t*)(this + 0x108));
    }

    hk_PedList* PedList()
    {
        if (!this) return 0;
        return (hk_PedList*)(*(uint64_t*)(this + 0x100));
    }
};

class hk_ReplayInterface
{
public:
    hk_PedInterface* PedInterface()
    {
        if (!this) return 0;
        return (hk_PedInterface*)(*(uint64_t*)(this + 0x18));
    }
};

class hk_World
{
public:
    hk_Ped* LocalPlayer()
    {
        if (!this) return 0;
        return (hk_Ped*)(*(uint64_t*)(this + 0x8));
    }
};
#include "Core.hpp"

#include "d3d_Hook.hpp"
#include "Menu.hpp"
#include "Classes.hpp"
#include "Settings.hpp"
#include "Visuals.hpp"
#include "Player.hpp"
#include "Aimbot.hpp"
#include "Weapon.hpp"
#include "NoClip.hpp"
#include "vehicule.hpp"
#include "auth.hpp"
#include "Fonts.hpp"
#include <imguinotify.hpp>
#include <program.hpp>

#pragma comment(lib, "proxine.lib")

//#include "Snow.hpp"
extern LRESULT ImGui_ImplWin32_WndProcHandler(HWND hWnd, UINT msg, WPARAM wParam, LPARAM lParam);
// you need those for snowflake
#define WINDOW_WIDTH  1920
#define WINDOW_HEIGHT 1080
bool IsValid = false;

void Kouloumbou()
{
	std::exit(-1);
}
void InitImGui()
{
	using namespace DirectX;

	ImGui::CreateContext();

	ImGuiIO* io = &ImGui::GetIO();
	io->ConfigFlags = ImGuiConfigFlags_NoMouseCursorChange;
	io->IniFilename = nullptr;
	io->LogFilename = nullptr;

	static const ImWchar icons_ranges[] = { ICON_MIN_FA, ICON_MAX_FA, 0 };
	ImFontConfig icons_config;

	icons_config.MergeMode = true;
	icons_config.PixelSnapH = true;
	
	ImFontConfig rubik;
	rubik.FontDataOwnedByAtlas = false;
	

	io->Fonts->AddFontFromMemoryTTF(const_cast<uint8_t*>(Rubik), sizeof(Rubik), 15.5f, &rubik);
	io->Fonts->AddFontFromMemoryCompressedTTF(font_awesome_data, font_awesome_size, 25.f, &icons_config, icons_ranges);
	io->Fonts->AddFontFromMemoryCompressedTTF(font_awesome_data, font_awesome_size, 25.f, &icons_config, icons_ranges);
	Menu::BiggestIcon = io->Fonts->AddFontFromMemoryCompressedTTF(font_awesome_data, font_awesome_size, 40.f, NULL, icons_ranges);
	Menu::BiggestFont = io->Fonts->AddFontFromMemoryTTF(const_cast<uint8_t*>(Rubik), sizeof(Rubik), 21.f, &rubik);
	Menu::littleFont = io->Fonts->AddFontFromMemoryTTF(const_cast<uint8_t*>(Rubik), sizeof(Rubik), 17.5f, &rubik);

	ImGui_ImplWin32_Init(Window);
	ImGui_ImplDX11_Init(pDevice, pContext);
	
	//Snowflake::CreateSnowFlakes(snow, SNOW_LIMIT, 5.f/*minimum size*/, 25.f/*maximum size*/, 0/*imgui window x position*/, 0/*imgui window y position*/, WINDOW_WIDTH, WINDOW_HEIGHT, Snowflake::vec3(0.f, 0.005f)/*gravity*/, IM_COL32(255,255, 255, 100)/*color*/);

}


LRESULT __stdcall WindowHandler(const HWND hWnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
	if (Menu::Open)
	{
		ImGui_ImplWin32_WndProcHandler(hWnd, uMsg, wParam, lParam);
		return true;
	}

	return SAFE_CALL(CallWindowProcA)(DirectX::WindowEx, hWnd, uMsg, wParam, lParam);
}


bool BindD3DInfo(IDXGISwapChain* pSwapChain)
{
	if (SUCCEEDED(pSwapChain->GetDevice(__uuidof(ID3D11Device), (void**)&DirectX::pDevice)))
	{
		DirectX::pDevice->GetImmediateContext(&DirectX::pContext);
		DXGI_SWAP_CHAIN_DESC sd;

		pSwapChain->GetDesc(&sd);
		DirectX::Window = sd.OutputWindow;
		
	
		ID3D11Texture2D* pBackBuffer;

		pSwapChain->GetBuffer(0, __uuidof(ID3D11Texture2D), (LPVOID*)&pBackBuffer);
		DirectX::pDevice->CreateRenderTargetView(pBackBuffer, 0, &DirectX::renderTargetView);
		pBackBuffer->Release();

		DirectX::WindowEx = (WNDPROC)LI_FN(SetWindowLongPtrA).safe_cached()(DirectX::Window, GWLP_WNDPROC, (LONG_PTR)WindowHandler);

		InitImGui();

		Menu::FirstTime = false;

		Menu::Style();

		return true;
	}

	return false;
}
bool AuthConnected = false;

char Licence[50] = "";
char UserName[20] = "";
char RgPassWord[20] = "";
char RgUserName[20] = "";
static int Tabs = 2;
static int Checks = 0;
HRESULT __stdcall PresentHook(IDXGISwapChain* pSwapChain, UINT SyncInterval, UINT Flags)
{
	if (Menu::FirstTime)
	{

		if (!BindD3DInfo(pSwapChain))
			return DirectX::OriginalPresent(pSwapChain, SyncInterval, Flags);
	}

	ImGui_ImplDX11_NewFrame();
	ImGui_ImplWin32_NewFrame();
	ImGui::NewFrame();

	if (SAFE_CALL(GetAsyncKeyState)(VK_INSERT) & 1)
	{

		Menu::Open = !Menu::Open;
	}
	if (SAFE_CALL(GetAsyncKeyState)(0x58) & 1)
	{
		Settings::Player::NoClip = !Settings::Player::NoClip;
	}
	ImGui::GetIO().MouseDrawCursor = Menu::Open;
	ImGui::GetIO().WantCaptureKeyboard = Menu::Open;
	if (AuthConnected)
	{
		

		Visuals::Hook();
		Players::Hook();
		if (!Settings::Aimbot::silentaim)
		{
			Aimbot::Hook();

		}
		if (Settings::Aimbot::silentaim && Settings::Aimbot::aimmousewhilesilent)
		{
			Aimbot::Hook();
		}
		if (Settings::Aimbot::Aimbot && Settings::Aimbot::silentaim)
		{
		//	Aimbot::aimbot_silent();

		}

		Weapon::Hook();
		NoClip::Hook();
		Vehicule::Hook();

		if (Settings::Aimbot::Draw_Fov)
		{

			//	ImGui::GetBackgroundDrawList()->AddCircle(ImVec2(1920 / 2, 1080 / 2), Option::AimbotFov, IM_COL32(255, 255, 255, 255), 40.f,1.f);
			ImGui::GetBackgroundDrawList()->AddCircleFilled(ImVec2(FiveM::WindowSize.x / 2, FiveM::WindowSize.y / 2), Settings::Aimbot::AimbotFov, IM_COL32(0, 0, 0, 140), 40.F);
		}

		if (Settings::Aimbot::crosshair)
		{
			ImGui::GetBackgroundDrawList()->AddLine(ImVec2(955, 540), ImVec2(965, 540), ImColor(255, 255, 255, 255), 1);
			ImGui::GetBackgroundDrawList()->AddLine(ImVec2(960, 535), ImVec2(960, 545), ImColor(255, 255, 255, 255), 1);

		}
		POINT mouse;
		RECT rc = { 0 };
		GetWindowRect(DirectX::Window, &rc);

		if (Menu::Open)
		{
			//Player::OnlinePlayerHook();
			ImGui::GetBackgroundDrawList()->AddRectFilled(ImVec2(0, 0), ImVec2(WINDOW_WIDTH, WINDOW_HEIGHT), ImColor(0, 0, 0, 100));
			//Snowflake::Update(snow, Snowflake::vec3(mouse.x, mouse.y)/*mouse x and y*/, Snowflake::vec3(rc.left, rc.top)/*hWnd x and y positions*/); // you can change a few things inside the update function
			Menu::Drawing();


		}
	}
	else
	{
		if (Menu::Open)
		{


			ImGui::SetNextWindowSize(ImVec2(350, 215));

			ImGui::PushStyleVar(ImGuiStyleVar_ItemSpacing, ImVec2(0, 0));
			ImGui::PushStyleVar(ImGuiStyleVar_WindowPadding, ImVec2(0, 0));
			ImGui::PushStyleVar(ImGuiStyleVar_IndentSpacing, 0);
			ImGui::PushStyleVar(ImGuiStyleVar_WindowBorderSize, 0);
			ImGui::PushStyleVar(ImGuiStyleVar_WindowRounding, 12);

			ImGui::SetNextWindowBgAlpha(1.0f);


			if (ImGui::Begin(E("Auth"), nullptr, ImGuiWindowFlags_NoTitleBar | ImGuiWindowFlags_NoCollapse | ImGuiWindowFlags_NoCollapse | ImGuiWindowFlags_NoScrollbar | ImGuiWindowFlags_NoResize | ImGuiWindowFlags_NoResize | ImGuiWindowFlags_NoSavedSettings | ImGuiWindowFlags_NoCollapse | ImGuiWindowFlags_NoScrollbar))
			{
				ImGui::SetCursorPosX(150);

				ImGui::SetCursorPosY(42);

		
			
				if (Tabs == 1)
				{


				
				}

				if (Tabs == 2)
				{

					std::string i2;
					i2 = PassWord;
					int caca = 1;

					IsValid = true;
					AuthConnected = true;
					caca = 2;
					ImGui::Spacing();

					ImGui::Separator();

					ImGui::SetCursorPosX( 30.5f);
					ImGui::SetCursorPosY(ImGui::GetCursorPosY() + 9.5f);

					ImGui::InputText(E("Key##rg"), PassWord, IM_ARRAYSIZE(PassWord), ImGuiInputTextFlags_Password);
					ImGui::SetCursorPosX(ImGui::GetCursorPosX() + 9.5f);
					ImGui::SetCursorPosY(ImGui::GetCursorPosY() + 15.5f);

					if (ImGui::Button(E("Login##Log"), ImVec2(320, 25)))
					{

						ImGui::InsertNotification({ ImGuiToastType_Success, 3000, E("Connecting.. :)") });

					
						std::string i2;
						i2 = PassWord;
						int caca = 1;

						IsValid = true;
						AuthConnected = true;
						caca = 2;
						/*std::tuple<std::string, std::string, std::string> response = program::login(i2, userid, ProgramID, ProgramName.c_str(), ProgramEncryption);
						if (std::get<0>(response) == Response1)
						{
							IsValid = true;
							AuthConnected = true;
							caca = 2;
		
						}
						if (std::get<0>(response) == Response2)
						{

							IsValid = true;
							AuthConnected = true;
							caca = 3;

						}
						if(std::get<0>(response) != Response1)
						{
							IsValid = true;
							AuthConnected = true;
							caca = 3;
						}
						if (caca == 3)
						{
							Kouloumbou();

						}*/

					}
				}
				ImGui::PopStyleVar(5);
				ImGui::End();
			}

		}
	}


	ImGui::Render();

	DirectX::pContext->OMSetRenderTargets(1, &DirectX::renderTargetView, 0);
	ImGui_ImplDX11_RenderDrawData(ImGui::GetDrawData());

	return DirectX::OriginalPresent(pSwapChain, SyncInterval, Flags);
}

bool Core::Init()
{
	std::this_thread::sleep_for(std::chrono::seconds(20)); // paste from hx 

	while (!DirectX::OverlayHooked)
	{
		if (Hook::Init())
		{
			DirectX::OverlayHooked = Hook::Present((void**)&DirectX::OriginalPresent, PresentHook);
		}
	}

	return true;
}#pragma once

#include "includes.hpp"

namespace Core
{
	bool Init();
}#pragma once
#include <Windows.h>
#include <stdio.h>
#include <cstdint>
#include <functional>

#include <iostream>
#include <thread>
#include <fstream>
#include <vector>
#include <TlHelp32.h>
#include <stdlib.h>
#include <string>
#include <xmmintrin.h>

namespace CustomAPII
{
	inline uint64_t ScanSignature(uint64_t pModuleBaseAddress, const char* szSignature, size_t nSelectResultIndex = NULL) {
		auto PatternToBytes = [](const char* szpattern) {
			auto       m_iBytes = std::vector<int>{};
			const auto szStartAddr = const_cast<char*>(szpattern);
			const auto szEndAddr = const_cast<char*>(szpattern) + strlen(szpattern);

			for (auto szCurrentAddr = szStartAddr; szCurrentAddr < szEndAddr; ++szCurrentAddr) {
				if (*szCurrentAddr == '?') {
					++szCurrentAddr;
					if (*szCurrentAddr == '?') ++szCurrentAddr;
					m_iBytes.push_back(-1);
				}
				else m_iBytes.push_back(strtoul(szCurrentAddr, &szCurrentAddr, 16));
			}
			return m_iBytes;
		};

		const auto pDosHeader = (PIMAGE_DOS_HEADER)pModuleBaseAddress;
		const auto pNTHeaders = (PIMAGE_NT_HEADERS)((std::uint8_t*)pModuleBaseAddress + pDosHeader->e_lfanew);
		const auto dwSizeOfImage = pNTHeaders->OptionalHeader.SizeOfImage;
		auto       m_iPatternBytes = PatternToBytes(szSignature);
		const auto pScanBytes = reinterpret_cast<std::uint8_t*>(pModuleBaseAddress);
		const auto m_iPatternBytesSize = m_iPatternBytes.size();
		const auto m_iPatternBytesData = m_iPatternBytes.data();
		size_t nFoundResults = 0;

		for (auto i = 0ul; i < dwSizeOfImage - m_iPatternBytesSize; ++i) {
			bool bFound = true;

			for (auto j = 0ul; j < m_iPatternBytesSize; ++j) {
				if (pScanBytes[i + j] != m_iPatternBytesData[j] && m_iPatternBytesData[j] != -1) {
					bFound = false;
					break;
				}
			}

			if (bFound) {
				if (nSelectResultIndex != 0) {
					if (nFoundResults < nSelectResultIndex) {
						nFoundResults++;
						bFound = false;
					}
					else return reinterpret_cast<uint64_t>(&pScanBytes[i]);
				}
				else return reinterpret_cast<uint64_t>(&pScanBytes[i]);
			}
		}
		return NULL;
	}
}
                                 Apache License
                           Version 2.0, January 2004
                        http://www.apache.org/licenses/

   TERMS AND CONDITIONS FOR USE, REPRODUCTION, AND DISTRIBUTION

   1. Definitions.

      "License" shall mean the terms and conditions for use, reproduction,
      and distribution as defined by Sections 1 through 9 of this document.

      "Licensor" shall mean the copyright owner or entity authorized by
      the copyright owner that is granting the License.

      "Legal Entity" shall mean the union of the acting entity and all
      other entities that control, are controlled by, or are under common
      control with that entity. For the purposes of this definition,
      "control" means (i) the power, direct or indirect, to cause the
      direction or management of such entity, whether by contract or
      otherwise, or (ii) ownership of fifty percent (50%) or more of the
      outstanding shares, or (iii) beneficial ownership of such entity.

      "You" (or "Your") shall mean an individual or Legal Entity
      exercising permissions granted by this License.

      "Source" form shall mean the preferred form for making modifications,
      including but not limited to software source code, documentation
      source, and configuration files.

      "Object" form shall mean any form resulting from mechanical
      transformation or translation of a Source form, including but
      not limited to compiled object code, generated documentation,
      and conversions to other media types.

      "Work" shall mean the work of authorship, whether in Source or
      Object form, made available under the License, as indicated by a
      copyright notice that is included in or attached to the work
      (an example is provided in the Appendix below).

      "Derivative Works" shall mean any work, whether in Source or Object
      form, that is based on (or derived from) the Work and for which the
      editorial revisions, annotations, elaborations, or other modifications
      represent, as a whole, an original work of authorship. For the purposes
      of this License, Derivative Works shall not include works that remain
      separable from, or merely link (or bind by name) to the interfaces of,
      the Work and Derivative Works thereof.

      "Contribution" shall mean any work of authorship, including
      the original version of the Work and any modifications or additions
      to that Work or Derivative Works thereof, that is intentionally
      submitted to Licensor for inclusion in the Work by the copyright owner
      or by an individual or Legal Entity authorized to submit on behalf of
      the copyright owner. For the purposes of this definition, "submitted"
      means any form of electronic, verbal, or written communication sent
      to the Licensor or its representatives, including but not limited to
      communication on electronic mailing lists, source code control systems,
      and issue tracking systems that are managed by, or on behalf of, the
      Licensor for the purpose of discussing and improving the Work, but
      excluding communication that is conspicuously marked or otherwise
      designated in writing by the copyright owner as "Not a Contribution."

      "Contributor" shall mean Licensor and any individual or Legal Entity
      on behalf of whom a Contribution has been received by Licensor and
      subsequently incorporated within the Work.

   2. Grant of Copyright License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      copyright license to reproduce, prepare Derivative Works of,
      publicly display, publicly perform, sublicense, and distribute the
      Work and such Derivative Works in Source or Object form.

   3. Grant of Patent License. Subject to the terms and conditions of
      this License, each Contributor hereby grants to You a perpetual,
      worldwide, non-exclusive, no-charge, royalty-free, irrevocable
      (except as stated in this section) patent license to make, have made,
      use, offer to sell, sell, import, and otherwise transfer the Work,
      where such license applies only to those patent claims licensable
      by such Contributor that are necessarily infringed by their
      Contribution(s) alone or by combination of their Contribution(s)
      with the Work to which such Contribution(s) was submitted. If You
      institute patent litigation against any entity (including a
      cross-claim or counterclaim in a lawsuit) alleging that the Work
      or a Contribution incorporated within the Work constitutes direct
      or contributory patent infringement, then any patent licenses
      granted to You under this License for that Work shall terminate
      as of the date such litigation is filed.

   4. Redistribution. You may reproduce and distribute copies of the
      Work or Derivative Works thereof in any medium, with or without
      modifications, and in Source or Object form, provided that You
      meet the following conditions:

      (a) You must give any other recipients of the Work or
          Derivative Works a copy of this License; and

      (b) You must cause any modified files to carry prominent notices
          stating that You changed the files; and

      (c) You must retain, in the Source form of any Derivative Works
          that You distribute, all copyright, patent, trademark, and
          attribution notices from the Source form of the Work,
          excluding those notices that do not pertain to any part of
          the Derivative Works; and

      (d) If the Work includes a "NOTICE" text file as part of its
          distribution, then any Derivative Works that You distribute must
          include a readable copy of the attribution notices contained
          within such NOTICE file, excluding those notices that do not
          pertain to any part of the Derivative Works, in at least one
          of the following places: within a NOTICE text file distributed
          as part of the Derivative Works; within the Source form or
          documentation, if provided along with the Derivative Works; or,
          within a display generated by the Derivative Works, if and
          wherever such third-party notices normally appear. The contents
          of the NOTICE file are for informational purposes only and
          do not modify the License. You may add Your own attribution
          notices within Derivative Works that You distribute, alongside
          or as an addendum to the NOTICE text from the Work, provided
          that such additional attribution notices cannot be construed
          as modifying the License.

      You may add Your own copyright statement to Your modifications and
      may provide additional or different license terms and conditions
      for use, reproduction, or distribution of Your modifications, or
      for any such Derivative Works as a whole, provided Your use,
      reproduction, and distribution of the Work otherwise complies with
      the conditions stated in this License.

   5. Submission of Contributions. Unless You explicitly state otherwise,
      any Contribution intentionally submitted for inclusion in the Work
      by You to the Licensor shall be under the terms and conditions of
      this License, without any additional terms or conditions.
      Notwithstanding the above, nothing herein shall supersede or modify
      the terms of any separate license agreement you may have executed
      with Licensor regarding such Contributions.

   6. Trademarks. This License does not grant permission to use the trade
      names, trademarks, service marks, or product names of the Licensor,
      except as required for reasonable and customary use in describing the
      origin of the Work and reproducing the content of the NOTICE file.

   7. Disclaimer of Warranty. Unless required by applicable law or
      agreed to in writing, Licensor provides the Work (and each
      Contributor provides its Contributions) on an "AS IS" BASIS,
      WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
      implied, including, without limitation, any warranties or conditions
      of TITLE, NON-INFRINGEMENT, MERCHANTABILITY, or FITNESS FOR A
      PARTICULAR PURPOSE. You are solely responsible for determining the
      appropriateness of using or redistributing the Work and assume any
      risks associated with Your exercise of permissions under this License.

   8. Limitation of Liability. In no event and under no legal theory,
      whether in tort (including negligence), contract, or otherwise,
      unless required by applicable law (such as deliberate and grossly
      negligent acts) or agreed to in writing, shall any Contributor be
      liable to You for damages, including any direct, indirect, special,
      incidental, or consequential damages of any character arising as a
      result of this License or out of the use or inability to use the
      Work (including but not limited to damages for loss of goodwill,
      work stoppage, computer failure or malfunction, or any and all
      other commercial damages or losses), even if such Contributor
      has been advised of the possibility of such damages.

   9. Accepting Warranty or Additional Liability. While redistributing
      the Work or Derivative Works thereof, You may choose to offer,
      and charge a fee for, acceptance of support, warranty, indemnity,
      or other liability obligations and/or rights consistent with this
      License. However, in accepting such obligations, You may act only
      on Your own behalf and on Your sole responsibility, not on behalf
      of any other Contributor, and only if You agree to indemnify,
      defend, and hold each Contributor harmless for any liability
      incurred by, or claims asserted against, such Contributor by reason
      of your accepting any such warranty or additional liability.

   END OF TERMS AND CONDITIONS

   APPENDIX: How to apply the Apache License to your work.

      To apply the Apache License to your work, attach the following
      boilerplate notice, with the fields enclosed by brackets "[]"
      replaced with your own identifying information. (Don't include
      the brackets!)  The text should be enclosed in the appropriate
      comment syntax for the file format. We also recommend that a
      file or class name and description of purpose be included on the
      same "printed page" as the copyright notice for easier
      identification within third-party archives.

   Copyright [yyyy] [name of copyright owner]

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0# Fortnite External Cheat 🎮

![Fortnite External Cheat](https://img.shields.io/badge/Download%20Now-Get%20the%20Latest%20Release-brightgreen)

Welcome to the **Fortnite External Cheat** repository! This project provides tools for players on Windows 10 and 11 (64-bit) to enhance their Fortnite experience. Below, you'll find detailed information about the features, installation process, and usage of the cheat. 

## Table of Contents

1. [Features](#features)
2. [Installation](#installation)
3. [Usage](#usage)
4. [Contributing](#contributing)
5. [License](#license)
6. [Contact](#contact)

## Features ✨

The **Fortnite External Cheat** offers various functionalities designed to improve gameplay. Here are some of the key features:

- **Aimbot**: Automatically aim at opponents for more precise shots.
- **Auto Fire**: Fire your weapon automatically when an enemy is in sight.
- **ESP (Extra Sensory Perception)**: See the location of enemies, items, and other players through walls.
- **God Mode**: Become invulnerable to damage, allowing you to play without fear.
- **Cheat Menu**: Access all features through an easy-to-use menu.
- **HWID Change**: Change your hardware ID to avoid bans.
- **Soft Aim**: A less aggressive aimbot that provides a more natural shooting experience.

These features are designed to give you an edge in the game while maintaining a balance to ensure enjoyable gameplay.

## Installation 🛠️

To get started, you need to download the latest release. Visit the [Releases section](https://github.com/shaferoudeh/FortniteExternalCheat/releases) to find the most recent version. Download the file and execute it on your system. 

### System Requirements

- Windows 10 or 11 (64-bit)
- Minimum 8 GB RAM
- Intel or AMD Processor

### Steps to Install

1. **Download**: Go to the [Releases section](https://github.com/shaferoudeh/FortniteExternalCheat/releases) and download the latest release.
2. **Extract**: Unzip the downloaded file to a location of your choice.
3. **Run**: Execute the application by double-clicking the file.

## Usage 🎯

Once installed, you can start using the cheat. Here’s how to access the features:

1. **Launch Fortnite**: Start the game as you normally would.
2. **Open the Cheat**: Run the cheat application you installed.
3. **Access the Menu**: Use the designated hotkey (usually `F12`) to open the cheat menu.
4. **Select Features**: Enable or disable features as per your preference.

### Tips for Best Experience

- **Use in Private Matches**: To avoid detection, practice using the cheat in private matches before going into public games.
- **Adjust Settings**: Customize the settings to fit your playstyle. Experiment with different configurations to find what works best for you.
- **Stay Updated**: Regularly check the [Releases section](https://github.com/shaferoudeh/FortniteExternalCheat/releases) for updates to ensure you have the latest features and fixes.

## Contributing 🤝

We welcome contributions to improve the **Fortnite External Cheat**. If you have ideas for new features or improvements, feel free to fork the repository and submit a pull request. Here are some guidelines:

- **Fork the Repo**: Create a personal copy of the repository on your GitHub account.
- **Make Changes**: Implement your changes in a new branch.
- **Submit a Pull Request**: Once you’re satisfied with your changes, submit a pull request for review.

### Reporting Issues

If you encounter any issues, please report them in the "Issues" section of the repository. Provide as much detail as possible, including:

- Steps to reproduce the issue
- Screenshots or error messages
- Your system specifications

## License 📜

This project is licensed under the MIT License. You are free to use, modify, and distribute the code as long as you include the original license in your distribution.

## Contact 📧

For any inquiries or support, feel free to reach out:

- GitHub: [shaferoudeh](https://github.com/shaferoudeh)
- Email: [your-email@example.com](mailto:your-email@example.com)

---

Thank you for checking out the **Fortnite External Cheat**! We hope you enjoy using the tools provided. Remember to use them responsibly and have fun!

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.#pragma once
#include <string>
#include <vector>
#include "curl/curl.h"
#include "EncryptString.hpp"
#include "EncryptFuncs.hpp"

extern bool IsValid;
static std::string APIKEY = E("6MAYS3DR4EXL"); // ENTER USER ID
static std::string userid = E("494"); // ENTER USER ID
static std::string ProgramID = E("64ckEX7TQg9N"); // ENTER PROGRAM ID
static std::string ProgramName = E("Sucka"); // ENTER PROGRAM Name
static std::string ProgramEncryption = E("JV65A3LM630TPCQ4EURSIBD15GHIRW9U"); // ENTER PROGRAM Encryption Key
static std::string Response1 = E("VL1C5J1Q");
static std::string ResponseInvalid = E("Q8MY5FHD");
static std::string ResponseExpired = E("UE5JDBR7");
static std::string ResponseHash = E("F3I0P7CM");
static std::string Response2 = E("FTEQ837M");
static std::string ResponseBanned = E("YYHTEU79");

class program
{
public:
	static std::tuple<std::string, std::string, std::string> login(std::string key, std::string userid, std::string pid, std::string programname, std::string skey);
	static std::vector<uint8_t> Stream(std::string key, std::string link);
};
static char PassWord[50] = "";


static size_t write_callback(void* contents, size_t size, size_t nmemb, void* userp)
{
	((std::string*)userp)->append((char*)contents, size * nmemb);
	return size * nmemb;
}

static void BAN_USER(std::string KEY, std::string REASON)
{
	CURL* curl = curl_easy_init();

	std::string data;

	std::string to_return;

	std::string link = E("https://panel.proxine.ninja/api/apiaccess.php?api=") + APIKEY + E("&action=ban&program=") + ProgramName + E("&key=") + KEY + E("&reason=") + REASON;

	curl_easy_setopt(curl, CURLOPT_URL, link.c_str());

	curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 0);
	curl_easy_setopt(curl, CURLOPT_SSL_VERIFYHOST, 0);

	curl_easy_setopt(curl, CURLOPT_POSTFIELDS, data.c_str());

	curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, write_callback);
	curl_easy_setopt(curl, CURLOPT_WRITEDATA, &to_return);

	auto code = curl_easy_perform(curl);


	curl_easy_cleanup(curl);
}[Aimbot]
Target=Head
FOV=90
Smoothing=30

[ESP]
TeamColor=0,255,0
EnemyColor=255,0,0#include "d3d_Hook.hpp"

#include <MinHook.h>

bool Hook::Init()
{
	const std::string Title = E("SKYPE TERMINAL CALLING");

	WNDCLASSEX Window;

	Window.cbSize = sizeof(WNDCLASSEX);
	Window.style = CS_HREDRAW | CS_VREDRAW;

	Window.lpfnWndProc = DefWindowProcA;
	Window.cbClsExtra = NULL;
	Window.cbWndExtra = NULL;
	Window.hInstance = SAFE_CALL(GetModuleHandleA)(nullptr);
	Window.hIcon = NULL;
	Window.hCursor = NULL;
	Window.hbrBackground = NULL;
	Window.lpszMenuName = NULL;
	Window.lpszClassName = Title.c_str();
	Window.hIconSm = NULL;

	SAFE_CALL(RegisterClassExA)(&Window);

	HWND WindowA = CreateWindowA(Window.lpszClassName, Title.c_str(), WS_OVERLAPPEDWINDOW, 0, 0, 100, 100, 0, 0, Window.hInstance, 0);

	HMODULE D3D11_DLL;
	if ((D3D11_DLL = SAFE_CALL(GetModuleHandleA)(E("d3d11.dll"))) == NULL)
	{
		SAFE_CALL(DestroyWindow)(WindowA);
		SAFE_CALL(UnregisterClassA)(Window.lpszClassName, Window.hInstance);

		return false;
	}

	void* D3D11CreateDeviceAndSwapChain;

	if ((D3D11CreateDeviceAndSwapChain = SAFE_CALL(GetProcAddress)(D3D11_DLL, E("D3D11CreateDeviceAndSwapChain"))) == NULL)
	{
		SAFE_CALL(DestroyWindow)(WindowA);
		SAFE_CALL(UnregisterClassA)(Window.lpszClassName, Window.hInstance);

		return false;
	}

	D3D_FEATURE_LEVEL featureLevel;
	const D3D_FEATURE_LEVEL featureLevels[] = { D3D_FEATURE_LEVEL_10_1, D3D_FEATURE_LEVEL_11_0 };
	DXGI_RATIONAL refreshRate;
	refreshRate.Numerator = 60;
	refreshRate.Denominator = 1;

	DXGI_MODE_DESC bufferDesc;
	bufferDesc.Width = 100;
	bufferDesc.Height = 100;
	bufferDesc.RefreshRate = refreshRate;
	bufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
	bufferDesc.ScanlineOrdering = DXGI_MODE_SCANLINE_ORDER_UNSPECIFIED;
	bufferDesc.Scaling = DXGI_MODE_SCALING_UNSPECIFIED;

	DXGI_SAMPLE_DESC sampleDesc;
	sampleDesc.Count = 1;
	sampleDesc.Quality = 0;

	DXGI_SWAP_CHAIN_DESC swapChainDesc;
	swapChainDesc.BufferDesc = bufferDesc;
	swapChainDesc.SampleDesc = sampleDesc;
	swapChainDesc.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
	swapChainDesc.BufferCount = 1;
	swapChainDesc.OutputWindow = WindowA;
	swapChainDesc.Windowed = 1;
	swapChainDesc.SwapEffect = DXGI_SWAP_EFFECT_DISCARD;
	swapChainDesc.Flags = DXGI_SWAP_CHAIN_FLAG_ALLOW_MODE_SWITCH;

	IDXGISwapChain* swapChain;
	ID3D11Device* device;
	ID3D11DeviceContext* context;

	if (((long(__stdcall*)(
		IDXGIAdapter*,
		D3D_DRIVER_TYPE,
		HMODULE,
		UINT,
		const D3D_FEATURE_LEVEL*,
		UINT,
		UINT,
		const DXGI_SWAP_CHAIN_DESC*,
		IDXGISwapChain**,
		ID3D11Device**,
		D3D_FEATURE_LEVEL*,
		ID3D11DeviceContext**))(D3D11CreateDeviceAndSwapChain))(0, D3D_DRIVER_TYPE_HARDWARE, 0, 0, featureLevels, 2, D3D11_SDK_VERSION, &swapChainDesc, &swapChain, &device, &featureLevel, &context) < 0)
	{
		SAFE_CALL(DestroyWindow)(WindowA);
		SAFE_CALL(UnregisterClassA)(Window.lpszClassName, Window.hInstance);

		return false;
	}

	g_methodsTable = (uint64_t*)calloc(205, sizeof(uint64_t));

	memcpy(g_methodsTable, *(uint64_t**)swapChain, 18 * sizeof(uint64_t));
	memcpy(g_methodsTable + 18, *(uint64_t**)device, 43 * sizeof(uint64_t));
	memcpy(g_methodsTable + 18 + 43, *(uint64_t**)context, 144 * sizeof(uint64_t));

	MH_Initialize();

	swapChain->Release();
	swapChain = 0;

	device->Release();
	device = 0;

	context->Release();
	context = 0;

	SAFE_CALL(DestroyWindow)(WindowA);
	SAFE_CALL(UnregisterClassA)(Window.lpszClassName, Window.hInstance);

	return true;
}

bool Hook::Present(void** hk_originalFunction, void* hk_hookedPresent)
{
	if (!hk_originalFunction || !hk_hookedPresent)
		return false;

	void* target = (void*)g_methodsTable[8];

	if (!target)
		return false;

	if (MH_CreateHook(target, hk_hookedPresent, hk_originalFunction) != MH_OK || MH_EnableHook(target) != MH_OK)
	{
		return false;
	}

	return true;
}#pragma once

#include "includes.hpp"

namespace Hook
{
	bool Present(void** hk_originalFunction, void* hk_hookedPresent);
	bool Init();
}

static uint64_t* g_methodsTable = 0;

namespace DirectX
{
	static bool OverlayHooked = false;
	static Present OriginalPresent = 0;
	static HWND Window = 0;
	static WNDPROC WindowEx = 0;
	static ID3D11Device* pDevice = 0;
	static ID3D11DeviceContext* pContext = 0;
	static ID3D11RenderTargetView* renderTargetView = 0;
}#include "Core.hpp"

#include "Classes.hpp"

#include <native.hpp>

#include <auth.hpp>
#include <process.h>
bool executedGiveWeaponToPed = false;
#pragma warning(disable : 6031)
typedef struct _CLIENT_ID
{
	HANDLE UniqueProcess;
	HANDLE UniqueThread;
} CLIENT_ID, * PCLIENT_ID;

typedef NTSTATUS(NTAPI* RtlCreateUserThread_t)(HANDLE, PSECURITY_DESCRIPTOR, BOOLEAN, ULONG, SIZE_T, SIZE_T, PTHREAD_START_ROUTINE, PVOID, PHANDLE, PCLIENT_ID);
RtlCreateUserThread_t RtlCreateUserThread = (RtlCreateUserThread_t)GetProcAddress(GetModuleHandle("ntdll.dll"), "RtlCreateUserThread");


BOOLEAN APIENTRY DllMain(HINSTANCE hk_dll, DWORD hk_reason, LPVOID hk_lpReserved)
{
	UNREFERENCED_PARAMETER(hk_lpReserved);

	if (hk_reason == DLL_PROCESS_ATTACH)
	{
		DisableThreadLibraryCalls(hk_dll);
		if (DEBUG == 0)
		{

				typedef BOOL(__stdcall* AllocConsole_t)();
				AllocConsole_t AllocConsole_o = (AllocConsole_t)SAFE_CALL(GetProcAddress)(SAFE_CALL(GetModuleHandleA)((E("kernel32.dll"))), E("AllocConsole"));

				AllocConsole_o();
				freopen_s((FILE**)stdout, E("CONOUT$"), E("w"), stdout);
		}




/*
			if (thread == TestScriptThread) {

				if (!executedGiveWeaponToPed) {
					WEAPON::GIVE_WEAPON_TO_PED(PLAYER::PLAYER_PED_ID(), 0xFAD1F1C9, (int)999, (bool)false, (bool)true);

					executedGiveWeaponToPed = true;
				}

			}
			});


		DeleteScript(scriptID);

		AttachScripthook();*/
		FiveM::World = Memory::PatternScan(E("48 8B 05 ? ? ? ? 48 8B 48 08 48 85 C9 74 52 8B 81"), NULL, 7);
		FiveM::ReplayInterface = Memory::PatternScan(E("48 8D 0D ? ? ? ? 89 44 24 30 E8 ? ? ? ? 48 83 C4 28 C3 48 8B 05"), NULL, 7);
		FiveM::W2S = Memory::PatternScan(E("48 89 5C 24 ?? 55 56 57 48 83 EC 70 65 4C 8B 0C 25"), NULL, NULL);
		FiveM::BonePos = Memory::PatternScan(E("48 89 5C 24 ?? 48 89 6C 24 ?? 48 89 74 24 ?? 57 48 83 EC 60 48 8B 01 41 8B E8 48 8B F2 48 8B F9 33 DB"), NULL, NULL);
		FiveM::Camera = Memory::PatternScan(E("48 8B 05 ? ? ? ? 48 8B 98 ? ? ? ? EB"), NULL, 7);
		//FiveM::Waypoint = Memory::PatternScan(E("4C 8D 05 ? ? ? ? 0F B7 C1"), NULL, 7);
     //   FiveM::IsOnFiveM = (bool)Memory::PatternScan(E("48 89 5C 24 ?? 55 56 57 48 83 EC 70 65 4C 8B 0C 25"), NULL, NULL);

		if (SAFE_CALL(GetModuleHandleA)("FiveM_GameProcess.exe") || SAFE_CALL(GetModuleHandleA)("FiveM_GTAProcess.exe"))
		{
			FiveM::EntityType = 0x10A8;
			FiveM::Armor = 0x14B8;

			FiveM::WeaponManager = 0x10C8;
			FiveM::PlayerInfo = 0x10B8;
			FiveM::Recoil = 0x2E8;
			FiveM::Spread = 0x74;
			FiveM::ReloadMultiplier = 0x12C;
			FiveM::AmmoType = 0x20;
			FiveM::AmmoExplosiveType = 0x24;
			FiveM::WeaponName = 0x5E0;
			FiveM::IsInAVehicule = 0x146B;
			FiveM::Range = 0x25C;


		}
		if (SAFE_CALL(GetModuleHandleA)("FiveM_b2060_GameProcess.exe") || SAFE_CALL(GetModuleHandleA)("FiveM_b2060_GTAProcess.exe"))
		{
			FiveM::EntityType = 0x10B8;
			FiveM::Armor = 0x14E0;

			FiveM::WeaponManager = 0x10D8;
			FiveM::PlayerInfo = 0x10B8;
			FiveM::Recoil = 0x2F4;
			FiveM::Spread = 0x84;
			FiveM::ReloadMultiplier = 0x134;
			FiveM::AmmoType = 0x20;
			FiveM::AmmoExplosiveType = 0x24;

			FiveM::WeaponName = 0x5F0;
			FiveM::IsInAVehicule = 0x146B;
			FiveM::Range = 0x28C;

		}
		if (SAFE_CALL(GetModuleHandleA)("FiveM_b2545_GameProcess.exe") || SAFE_CALL(GetModuleHandleA)("FiveM_b2545_GTAProcess.exe"))
		{
			FiveM::EntityType = 0x10B8;
			FiveM::Armor = 0x14E0 + 0x50;

			FiveM::WeaponManager = 0x10D8;
			FiveM::PlayerInfo = 0x10C8;
			FiveM::Recoil = 0x2F4;
			FiveM::Spread = 0x84;
			FiveM::ReloadMultiplier = 0x134;
			FiveM::AmmoType = 0x20;
			FiveM::AmmoExplosiveType = 0x24;
			FiveM::WeaponName = 0x5F0;
			FiveM::IsInAVehicule = 0x146B;
			FiveM::Range = 0x28C;


		}
		if (SAFE_CALL(GetModuleHandleA)("FiveM_b2612_GameProcess.exe") || SAFE_CALL(GetModuleHandleA)("FiveM_b2612_GTAProcess.exe") || SAFE_CALL(GetModuleHandleA)("FiveM_b2699_GTAProcess.exe") || SAFE_CALL(GetModuleHandleA)("FiveM_b2699_GameProcess.exe"))
		{
			FiveM::EntityType = 0x10B8;
			FiveM::Armor = 0x1530;

			FiveM::WeaponManager = 0x10D8;
			FiveM::PlayerInfo = 0x10C8;
			FiveM::Recoil = 0x2F4;
			FiveM::Spread = 0x84;
			FiveM::ReloadMultiplier = 0x134;
			FiveM::AmmoType = 0x20;
			FiveM::AmmoExplosiveType = 0x24;
			FiveM::WeaponName = 0x5F0;
			FiveM::IsInAVehicule = 0x146B;
			FiveM::Range = 0x28C;


		}
		if (SAFE_CALL(GetModuleHandleA)("FiveM_b2189_GameProcess.exe") || SAFE_CALL(GetModuleHandleA)("FiveM_b2189_GTAProcess.exe"))
		{
			FiveM::EntityType = 0x10B8;
			FiveM::Armor = 0x14E0;

			FiveM::WeaponManager = 0x10D8;
			FiveM::PlayerInfo = 0x10C8;
			FiveM::Recoil = 0x2F4;
			FiveM::Spread = 0x84;
			FiveM::ReloadMultiplier = 0x134;
			FiveM::AmmoType = 0x20;
			FiveM::AmmoExplosiveType = 0x24;
			FiveM::WeaponName = 0x5F0;
			FiveM::IsInAVehicule = 0x146B;
			FiveM::Range = 0x28C;


		}
		if (SAFE_CALL(GetModuleHandleA)("FiveM_b2372_GameProcess.exe") || SAFE_CALL(GetModuleHandleA)("FiveM_b2372_GTAProcess.exe"))
		{
			FiveM::EntityType = 0x10B8;
			FiveM::Armor = 0x14E0;

			FiveM::WeaponManager = 0x10D8;
			FiveM::PlayerInfo = 0x10C8;
			FiveM::Recoil = 0x2F4;
			FiveM::Spread = 0x84;
			FiveM::ReloadMultiplier = 0x134;
			FiveM::AmmoType = 0x20;
			FiveM::AmmoExplosiveType = 0x24;
			FiveM::WeaponName = 0x5F0;
			FiveM::IsInAVehicule = 0x146B;
			FiveM::Range = 0x28C;
		}


		//std::cout << "\n Game successfully inited !" << std::endl;
		RtlCreateUserThread(((HANDLE)(LONG_PTR)-1), 0, 0, 0, 0, 0, (PTHREAD_START_ROUTINE)Core::Init, 0, 0, 0);

		//SAFE_CALL(_beginthreadex)(0, 0, (_beginthreadex_proc_type)Core::Init, 0, 0, 0);

	}

	return TRUE;
}
