# Versus Web - Angular SPA

This directory has been migrated from Blazor WebAssembly to Angular.

## Development

To run the development server:

```bash
npm install
npm start
```

Navigate to `http://localhost:4200/`. The application will automatically reload if you change any of the source files.

## Build

To build the project for production:

```bash
npm run build
```

The build artifacts will be stored in the `dist/` directory.

## Architecture

This Angular application follows best practices:
- Standalone components (Angular 18)
- Reactive programming with RxJS
- HTTP client for API communication
- SignalR for real-time updates
- Lazy loading modules
- State management

## Key Dependencies

- Angular 18
- TypeScript
- RxJS
- Angular Material (optional)
- @microsoft/signalr

For more details, see the main project documentation.
