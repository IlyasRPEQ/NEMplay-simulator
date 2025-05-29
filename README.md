├── app/
│   ├── bidding/
│   │   └── rule_based.py          # Simple threshold strategy
│   ├── simulation/
│   │   └── simulate_dispatch.py   # Runs asset vs market simulation
│   ├── forecasts/
│   │   └── price_forecast.py      # Basic Prophet-based forecaster
│   ├── reports/
│   │   └── revenue_report.py      # Generates CSV/PDF output
│   └── models/
│       └── asset.py               # Battery, PV asset classes
├── api/
│   ├── routes/
│   │   └── simulate.py            # FastAPI route: /simulate
│   └── main.py                    # FastAPI app runner
├── data/
│   ├── raw/
│   └── processed/
├── db/
│   └── schemas.py                 # Pydantic models
├── frontend/                      # Optional Streamlit app
│   └── dashboard.py
├── utils/
│   └── helpers.py
├── tests/
│   └── test_simulation.py
├── requirements.txt
└── README.md# NEMplay-simulator
rom fastapi import APIRouter, HTTPException
from pydantic import BaseModel
from app.simulation.simulate_dispatch import simulate_dispatch

router = APIRouter()

class SimulationRequest(BaseModel):
    asset_type: str
    capacity_mw: float
    soc_start: float
    price_data: list[float]  # 5-minute prices for a day
    strategy: str

@router.post("/simulate")
def run_simulation(request: SimulationRequest):
    try:
        result = simulate_dispatch(
            asset_type=request.asset_type,
            capacity_mw=request.capacity_mw,
            soc_start=request.soc_start,
            price_data=request.price_data,
            strategy=request.strategy
        )
        return {"status": "success", "result": result}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))


def simulate_dispatch(asset_type, capacity_mw, soc_start, price_data, strategy):
    soc = soc_start
    revenue = 0
    dispatch_log = []

    for t, price in enumerate(price_data):
        dispatch = 0
        if strategy == "simple_threshold":
            if price > 100:  # Example threshold
                dispatch = min(soc, capacity_mw * 0.0833)  # 5-min dispatch
                soc -= dispatch
                revenue += dispatch * price
        dispatch_log.append({"interval": t, "price": price, "dispatch_mw": dispatch, "soc": soc})

    return {
        "total_revenue": revenue,
        "final_soc": soc,
        "dispatch_log": dispatch_log
    }

def simulate_dispatch(asset_type, capacity_mw, soc_start, price_data, strategy):
    soc = soc_start
    revenue = 0
    dispatch_log = []

    for t, price in enumerate(price_data):
        dispatch = 0
        if strategy == "simple_threshold":
            if price > 100:  # Example threshold
                dispatch = min(soc, capacity_mw * 0.0833)  # 5-min dispatch
                soc -= dispatch
                revenue += dispatch * price
        dispatch_log.append({"interval": t, "price": price, "dispatch_mw": dispatch, "soc": soc})

    return {
        "total_revenue": revenue,
        "final_soc": soc,
        "dispatch_log": dispatch_log
    }


from fastapi import FastAPI
from api.routes.simulate import router as simulate_router

app = FastAPI(
    title="NEMPlay Simulator API",
    description="Optimizes and simulates dispatch in the Australian NEM",
    version="0.1"
)

app.include_router(simulate_router, prefix="/api")

fastapi
uvicorn
pydantic
pandas
prophet

uvicorn api.main:app --reload

NEM Automate
