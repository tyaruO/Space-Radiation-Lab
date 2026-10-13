# Space-Radiation-Lab
思考実験

"""
Space Radiation Lab
宇宙放射線遮蔽・エネルギー回収・滞在時間の概念シミュレーター

注意:
- 学習・概念検討用の簡易モデルです。
- 遮蔽係数はデモ用の仮定値であり、実測値ではありません。
- 実際の宇宙放射線輸送、二次粒子、人体線量などを正確に再現しません。
- 出力を宇宙飛行の安全判断に使用しないでください。
"""

import argparse
import json
import math
import random
from datetime import datetime, timezone


# --------------------------------------------------
# 1. 放射線遮蔽の概念計算
# --------------------------------------------------

def calculate_shielding(
    baseline_dose_rate,
    water_cm,
    polymer_cm,
    water_coefficient=0.025,
    polymer_coefficient=0.030,
):
    """
    仮定した指数減衰モデルで概念上の透過率を計算する。

    baseline_dose_rate:
        遮蔽前の想定線量率 (mSv/day)

    water_cm:
        水の厚さ (cm)

    polymer_cm:
        ポリエチレン相当材の厚さ (cm)

    係数はデモ用。実際の遮蔽性能を示すものではない。
    """

    values = [
        baseline_dose_rate,
        water_cm,
        polymer_cm,
        water_coefficient,
        polymer_coefficient,
    ]

    if any(not math.isfinite(v) or v < 0 for v in values):
        raise ValueError("入力値は有限の0以上の数値にしてください。")

    optical_depth = (
        water_coefficient * water_cm
        + polymer_coefficient * polymer_cm
    )

    transmission = math.exp(-optical_depth)

    estimated_dose_rate = baseline_dose_rate * transmission

    return {
        "water_thickness_cm": water_cm,
        "polymer_thickness_cm": polymer_cm,
        "transmission_fraction": transmission,
        "toy_reduction_percent": (1 - transmission) * 100,
        "toy_dose_rate_mSv_day": estimated_dose_rate,
        "model_warning": (
            "デモ用の指数減衰モデル。実際の宇宙放射線遮蔽性能ではありません。"
        ),
    }


# --------------------------------------------------
# 2. エネルギー回収の概算
# --------------------------------------------------

def calculate_energy_recovery(
    incident_flux_W_m2=0.0,
    absorbed_fraction=0.0,
    thermal_fraction=1.0,
    conversion_efficiency=0.05,
):
    """
    仮定したエネルギー流束から電力密度を計算する。

    incident_flux_W_m2:
        入射エネルギー流束 (W/m²)

    absorbed_fraction:
        吸収割合 (0～1)

    thermal_fraction:
        吸収エネルギーのうち利用可能な熱になる割合 (0～1)

    conversion_efficiency:
        熱から電気への変換効率 (0～1)
    """

    if not math.isfinite(incident_flux_W_m2) or incident_flux_W_m2 < 0:
        raise ValueError("エネルギー流束は0以上にしてください。")

    fractions = [
        absorbed_fraction,
        thermal_fraction,
        conversion_efficiency,
    ]

    if any(
        not math.isfinite(v) or not 0 <= v <= 1
        for v in fractions
    ):
        raise ValueError("各割合・効率は0から1の範囲にしてください。")

    electrical_power = (
        incident_flux_W_m2
        * absorbed_fraction
        * thermal_fraction
        * conversion_efficiency
    )

    return {
        "incident_flux_W_m2": incident_flux_W_m2,
        "absorbed_fraction": absorbed_fraction,
        "thermal_fraction": thermal_fraction,
        "conversion_efficiency": conversion_efficiency,
        "estimated_electrical_power_W_m2": electrical_power,
        "warning": (
            "入力値に基づく概算です。宇宙放射線から実際に得られる電力を"
            "予測するには、根拠のあるエネルギー流束と実験データが必要です。"
        ),
    }


# --------------------------------------------------
# 3. 線量予算に基づく滞在時間の算術計算
# --------------------------------------------------

def calculate_stay_time(dose_budget_mSv, dose_rate_mSv_day):
    """
    線量予算 ÷ 想定線量率による単純計算。

    これは安全な滞在可能時間を示すものではない。
    """

    values = [dose_budget_mSv, dose_rate_mSv_day]

    if any(not math.isfinite(v) or v < 0 for v in values):
        raise ValueError("線量の入力値は有限の0以上にしてください。")

    if dose_rate_mSv_day == 0:
        days = None
        hours = None
    else:
        days = dose_budget_mSv / dose_rate_mSv_day
        hours = days * 24

    return {
        "illustrative_dose_budget_mSv": dose_budget_mSv,
        "assumed_dose_rate_mSv_day": dose_rate_mSv_day,
        "arithmetic_days": days,
        "arithmetic_hours": hours,
        "warning": (
            "算術上のシナリオであり、安全な滞在時間や推奨被ばく限度ではありません。"
        ),
    }


# --------------------------------------------------
# 4. 模擬センサー
# --------------------------------------------------

def read_simulated_sensor():
    """
    テスト用の模擬データを生成する。
    実際の放射線センサーには接続しない。
    """

    return {
        "timestamp_utc": datetime.now(timezone.utc).isoformat(),
        "simulated_dose_rate_uSv_h": round(
            random.uniform(0.05, 2.0), 4
        ),
        "simulated_temperature_C": round(
            random.uniform(-10.0, 35.0), 2
        ),
        "source": "SIMULATED_DATA_NOT_REAL_MEASUREMENTS",
    }


# --------------------------------------------------
# 5. 統合シミュレーション
# --------------------------------------------------

def run_simulation(
    water_cm=10.0,
    polymer_cm=10.0,
    baseline_dose_rate=1.3,
    dose_budget_mSv=600.0,
    incident_flux_W_m2=0.0,
    absorbed_fraction=0.0,
    thermal_fraction=1.0,
    conversion_efficiency=0.05,
):
    """
    遮蔽・エネルギー回収・滞在時間・模擬センサーを統合する。
    """

    shielding = calculate_shielding(
        baseline_dose_rate=baseline_dose_rate,
        water_cm=water_cm,
        polymer_cm=polymer_cm,
    )

    energy = calculate_energy_recovery(
        incident_flux_W_m2=incident_flux_W_m2,
        absorbed_fraction=absorbed_fraction,
        thermal_fraction=thermal_fraction,
        conversion_efficiency=conversion_efficiency,
    )

    stay_time = calculate_stay_time(
        dose_budget_mSv=dose_budget_mSv,
        dose_rate_mSv_day=shielding["toy_dose_rate_mSv_day"],
    )

    sensor = read_simulated_sensor()

    return {
        "project": "Space Radiation Lab",
        "version": "0.1.0",
        "disclaimer": (
            "研究・学習用の概念モデル。実際の宇宙機設計や安全判断には使用不可。"
        ),
        "shielding_scenario": shielding,
        "energy_recovery_scenario": energy,
        "stay_time_arithmetic": stay_time,
        "simulated_sensor": sensor,
    }


# --------------------------------------------------
# 6. コマンドライン実行
# --------------------------------------------------

def main():
    parser = argparse.ArgumentParser(
        description="Space Radiation Lab: 統合概念シミュレーター"
    )

    parser.add_argument("--water-cm", type=float, default=10.0)
    parser.add_argument("--polymer-cm", type=float, default=10.0)
    parser.add_argument(
        "--baseline-dose-rate",
        type=float,
        default=1.3,
        help="仮定した遮蔽前線量率 (mSv/day)",
    )
    parser.add_argument("--dose-budget-mSv", type=float, default=600.0)
    parser.add_argument("--incident-flux-W-m2", type=float, default=0.0)
    parser.add_argument("--absorbed-fraction", type=float, default=0.0)
    parser.add_argument("--thermal-fraction", type=float, default=1.0)
    parser.add_argument("--conversion-efficiency", type=float, default=0.05)
    parser.add_argument("--output", type=str, default="report.json")

    args = parser.parse_args()

    report = run_simulation(
        water_cm=args.water_cm,
        polymer_cm=args.polymer_cm,
        baseline_dose_rate=args.baseline_dose_rate,
        dose_budget_mSv=args.dose_budget_mSv,
        incident_flux_W_m2=args.incident_flux_W_m2,
        absorbed_fraction=args.absorbed_fraction,
        thermal_fraction=args.thermal_fraction,
        conversion_efficiency=args.conversion_efficiency,
    )

    output = json.dumps(
        report,
        ensure_ascii=False,
        indent=2,
    )

    with open(args.output, "w", encoding="utf-8") as file:
        file.write(output + "\n")

    print(output)
    print(f"\n結果を {args.output} に保存しました。")


if __name__ == "__main__":
    main()

python space_radiation_lab.py

python space_radiation_lab.py --water-cm 20 --polymer-cm 15
