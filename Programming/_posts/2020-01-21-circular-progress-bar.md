---
layout: post
title: Using Skia Sharp to make a circular progress bar
published: true
thumb: assets/images/blog/blog-post-thumb-7.jpg
---

Create a simple rounded progress indicator using only SkiaSharp. No need for native renderers. 

### What we're going to build:

A beautiful reusable control that indicates progress to your users. I'll show you how to add rounded ends and how you can get started using animations to create sleek transations from one state to another.

[See here on GitHub](https://github.com/RobertBickers/xamarin-ui-controls/blob/main/Xamarin.UI.Controls.Demo/Codetreehouse.Xamarin.UI.Controls/RadialProgressIndicator/RadialProgressIndicator.cs)

The approach here is to create a reusable `ContentView` that can be included on a page much like any other control.

Using a bindable property for Progress you can control the ring. This is great for showing progress as an action completes as a custom loading spinner, or as an overall progress for a series of user actions.

The following properties have been exposed, but you can of course add more.

- StrokeWidth
- BackgroundRingColour
- ProgressColour
- Progress


## Create a new ContentView

I've called it, `RadialProgressIndicator`. There isn't any Xaml required, so you can create a code only ContentView. We'll come back to this file later to fill it in


## Place the new control on the page

Reference the namespace within the page that it will be used, by including this at the top of the xml namespace declaration:

	xmlns:controls="clr-namespace:Codetreehouse.Xamarin.UI.Controls;assembly=Codetreehouse.Xamarin.UI.Controls"

You'll need to update these to meet the correct placement within your own app. For me, this is because my new control is in the `Codetreehouse.Xamarin.UI.Controls` namespace, within a separate library called `Codetreehouse.Xamarin.UI.Controls`.

	<controls:RadialProgressIndicator
    	HorizontalOptions="Center"
		HeightRequest="150"
		WidthRequest="150"
		StrokeWidth="40"
		Progress="{Binding Progress}"
		BackgroundRingColour="White"
		ProgressColour="DarkBlue"/>
        

## Include the SkiaSharp.Forms NuGet Package

Install the `SkiaSharp.Views.Forms` NuGet package

[See SkiaSharp.Views.Forms in NuGet Gallery](https://www.nuget.org/packages/SkiaSharp.Views.Forms/)


## Fill in the control

	using System;
	using SkiaSharp;
	using SkiaSharp.Views.Forms;
	using Xamarin.Forms;

    namespace Codetreehouse.Xamarin.UI.Controls
    {
        public class RadialProgressIndicator : SKCanvasView
        {
            public class Circle
            {
                private readonly Func<SKImageInfo, SKPoint> _centrePlacementFunc;

			public Circle(Func<SKImageInfo, SKPoint> centrePlacementFunction)
			{
				_centrePlacementFunc = centrePlacementFunction;
			}

			public SKPoint Center { get; set; }

			public float Radius { get; set; }

			public SKRect Rect => new SKRect(Center.X - Radius, Center.Y - Radius, Center.X + Radius, Center.Y + Radius);


			public void CalculateCenter(SKImageInfo argsInfo, float strokeThickness)
			{

				Radius = (argsInfo.Width / 2) - strokeThickness;
				Center = _centrePlacementFunc(argsInfo);
			}
		}

		Circle _circle;

		public RadialProgressIndicator()
		{
			_circle = new Circle((info) => new SKPoint((float)info.Width / 2, (float)info.Height / 2));
		}

		protected override void OnPaintSurface(SKPaintSurfaceEventArgs paintSurfaceEventArgs)
		{
			_circle.CalculateCenter(paintSurfaceEventArgs.Info, StrokeWidth);

			paintSurfaceEventArgs.Surface.Canvas.Clear();

			DrawBackgroundCircle(paintSurfaceEventArgs.Surface.Canvas, _circle, StrokeWidth, BackgroundRingColour.ToSKColor());
			DrawProgressArc(paintSurfaceEventArgs.Surface.Canvas, _circle, Progress, StrokeWidth, ProgressColour.ToSKColor());
		}

		void DrawBackgroundCircle(SKCanvas canvas, Circle circle, float strokewidth, SKColor color)
		{
			canvas.DrawCircle(circle.Center, circle.Radius,
				new SKPaint()
				{
					StrokeWidth = strokewidth,
					Color = color,
					IsStroke = true
				});
		}

		void DrawProgressArc(SKCanvas canvas, Circle circle, int progress, float strokewidth, SKColor color)
		{
			var sweepAngle = progress * 3.6f;

			canvas.DrawArc(circle.Rect, 270, (float)sweepAngle, false,
				new SKPaint()
				{
					StrokeWidth = strokewidth,
					Color = color,
					IsStroke = true,
					StrokeCap = SKStrokeCap.Round
				});
		}

		public static readonly BindableProperty StrokeWidthProperty = BindableProperty.Create(nameof(StrokeWidth), typeof(int), typeof(RadialProgressIndicator), defaultValue: 10, propertyChanged: OnBindablePropertyChanged);

		public static readonly BindableProperty ProgressProperty = BindableProperty.Create(nameof(Progress), typeof(int), typeof(RadialProgressIndicator), propertyChanged: OnBindablePropertyChanged);

		public static readonly BindableProperty ProgressColourProperty = BindableProperty.Create(nameof(ProgressColour), typeof(Color), typeof(RadialProgressIndicator), defaultValue: Color.DarkGreen, propertyChanged: OnBindablePropertyChanged);

		public static readonly BindableProperty BackgroundRingColourProperty = BindableProperty.Create(nameof(ProgressColour), typeof(Color), typeof(RadialProgressIndicator), defaultValue: Color.LightGray, propertyChanged: OnBindablePropertyChanged);

		public int StrokeWidth
		{
			get { return (int)GetValue(StrokeWidthProperty); }
			set { SetValue(StrokeWidthProperty, value); }
		}

		public int Progress
		{
			get { return (int)GetValue(ProgressProperty); }
			set { SetValue(ProgressProperty, value); }
		}

		public Color ProgressColour
		{
			get { return (Color)GetValue(ProgressColourProperty); }
			set { SetValue(ProgressColourProperty, value); }
		}

		public Color BackgroundRingColour
		{
			get { return (Color)GetValue(BackgroundRingColourProperty); }
			set { SetValue(BackgroundRingColourProperty, value); }
		}


		static void OnBindablePropertyChanged(BindableObject bindable, object oldValue, object newValue)
			=> InvalidateDraw((RadialProgressIndicator)bindable);

		static void InvalidateDraw(RadialProgressIndicator progressCanvasView)
		{
			var context = progressCanvasView;
			context.InvalidateSurface();
		}
	}
	}












