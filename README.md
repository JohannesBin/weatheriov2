# Weather.io

Imagine building a weather app that feels more like a living, breathing window to the world's atmospheric conditions. That's the core idea behind Weather.io. The magic starts with a simple user input – typing in a location – and transforms into a rich, dynamic weather experience.

## The API Tango 🕺💃

Our first dance partner is the Mapbox Geocoding API. When a user types "San Francisco", we need to convert that human-readable location into precise geographic coordinates. Here's how our `fetchWeather` method does the heavy lifting:

```typescript
const fetchWeather = async (query: string) => {
  // Geocode the location
  const geocodeResponse = await fetch(
    `https://api.mapbox.com/geocoding/v5/mapbox.places/${encodeURIComponent(query)}.json?access_token=${MAPBOX_TOKEN}`
  );
  
  const geocodeData = await geocodeResponse.json();
  const [lon, lat] = geocodeData.features[0].center;
  
  // Fetch weather data
  const weatherResponse = await fetch(
    `${CORS_PROXY}https://api.met.no/weatherapi/locationforecast/2.0/compact?lat=${lat}&lon=${lon}`,
    {
      headers: {
        'User-Agent': 'Weather.io/1.0 github.com/weather-io'
      }
    }
  );
  
  const weatherData = await weatherResponse.json();
};
```

The real challenge? Dealing with the complexities of API interactions. We use a CORS proxy to sidestep cross-origin restrictions, and carefully handle potential errors. Each API call is a potential point of failure, so robust error handling is crucial.

## Type safety and data modeling

TypeScript becomes our best friend in managing complex weather data. Our `MetNoWeatherData` interface is a blueprint that ensures every piece of weather information is precisely typed:

```typescript
export interface MetNoWeatherData {
  properties: {
    timeseries: Array<{
      time: string;
      data: {
        instant: {
          details: {
            air_temperature: number;
            wind_speed: number;
          }
        };
        next_1_hours?: {
          summary: { symbol_code: string };
        };
      };
    }>;
  };
}
```

## UI Magic

The `CurrentWeather` component is where data transforms into experience. We dynamically generate background gradients based on weather conditions:

```typescript
const getWeatherGradient = () => {
  if (isRaining) return 'from-blue-900/30 via-slate-900/30 to-slate-900/30';
  if (isSunny) return 'from-amber-500/20 via-orange-500/10 to-slate-900/30';
  // More weather conditions...
};
```

Each weather state gets its own visual personality – rain brings subtle droplet effects, sunny days create warm, pulsing light backgrounds.

## State management

Our custom `useWeatherData` hook encapsulates the entire data fetching and state management logic:

```typescript
export function useWeatherData() {
  const [weatherData, setWeatherData] = useState<MetNoWeatherData | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const fetchWeather = async (query: string) => {
    setIsLoading(true);
    try {
      // Geocoding and weather fetching logic
    } catch (err) {
      setError(err.message);
    } finally {
      setIsLoading(false);
    }
  };

  return { weatherData, isLoading, error, fetchWeather };
}
```

This hook is a Swiss Army knife – managing loading states, error handling, and data fetching in a clean, reusable package.

## The bigger picture

Weather.io isn't just an app; it's a carefully orchestrated system. React provides the responsive UI, TypeScript ensures type safety, Firebase handles authentication, and a network of APIs brings global weather data to your fingertips. 

Every line of code is a bridge between raw meteorological data and a user-friendly, visually stunning experience. From the moment you type a location to seeing a richly detailed weather display, Weather.io transforms complex data into something intuitive and beautiful.

It's technology and nature, talking to each other – with style.
